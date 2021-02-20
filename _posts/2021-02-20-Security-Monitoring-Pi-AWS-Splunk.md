---
published: false
title: Pi + AWS = Security Monitoring
author: f3dai
image: 'https://imgur.com/55RpuAS.png'
---
An effective security monitoring system using a Pi. Similar concept applies for any sized organisation with better hardware and more complex network, so scale as you wish.

**Requirements:**

- A Raspberry Pi, I am using a Pi 4 for this article. I will also discuss for other scenarios.
- A managed switch capable of mirroring (optional - I'll speak about this in the topology section).
- Splunk Enterprise and splunk forwarder.
- Either an instance running on any cloud provider, a virtual machine, or similar. I will be using my AWS educate account. If this is for actual production, use an actual server that is always running. For example, and ESXi server if you choose to utilise a VM.

## Topology

This will be the topology of my security monitoring system given, if we are using a Pi which has wireless AP capabilities, and a conventional home router:

![Pi 4 topology](https://imgur.com/MwHt3ge.png)

If you don't have a Pi that can serve as an AP, you can just put an AP in the switch. Or don't, but anything connected to the switch or Pi will be monitored. For example, if you have your hosts connected via the router, you wont be able to monitor it.

![Pi Additional AP topology](https://imgur.com/TryyyNk.png)

If you only want to monitor wireless devices, or you don't want to purchase a switch, you can just connect the Pi directly to the router, without any need for port mirroring:

![Pi No Switch topology](https://imgur.com/fYmZO0d.png)

## Raspberry Pi 4

![pi logo](https://imgur.com/eIDcULe.png)

I'm using a Pi 4 with Raspian installed. If you haven't setup the OS on your Pi yet, you'll need to use the Raspberry Pi imager to burn your desired OS on the micro SD card. Find download links here:

[Raspberry Pi Software - Downloads](https://www.raspberrypi.org/software/)

Not going to cover booting up the OS for the first time. Setup headless / connect to monitor, whatever floats your boat.

Let's start off by installing an (N)IDS. You have quite a few to choose from and at this point it comes down to personal preference. Suricata is probably my favourite, I have also tried Snort and Zeek. For this article I will use Zeek. This part will vary depending on your OS obviously.

### Zeek Installation

I always like to install by source, you can see if there are any repositories and install by package but for the sake of good practice I will show you how to install Zeek by source. Why is it better to install by source? You have more control, trim down features and you can familiarise yourself with how it works by seeing how it is installed. 

[Zeek Documentation - Installation](https://docs.zeek.org/en/current/install/install.html)


Dependencies:

- Cmake 2.8.12 (or greater)
- Make
- C/C++ Compiler with C++ 11 support (GCC 4.8+ or Clang 3.3+)
- SWIG
- Bison 2.5 or greater
- Flex
- Libpcap headers
- OpenSSL headers
- zlib headers
- Python

<pre>sudo apt install cmake make gcc g++ flex bison libpcap-dev libssl-dev python-dev swig zlib1g-dev</pre>


Some extra packages - geolocation, e-mail sending, curl for http and git for downloading zeek files:

<pre>sudo apt install libmaxminddb-dev postfix curl git</pre>

Now let's get the installation file. You can download these into your home directory.

<pre>git clone --recursive https://github.com/zeek/zeek
cd zeek</pre>

Now install with these commands:

<pre>./configure
make</pre>

This will take a very long time. Took me quite a few hours so I left it overnight. Once this has completed, run the following:

<pre>sudo make install</pre>

Zeek will now be installed in _/usr/local/zeek_ so we can add the bin location to our PATH by adding this to **~/.profile**:

<pre>export PATH=/usr/local/zeek/bin:$PATH</pre>

Not going to go too much into configuration and where all the files are. You can search that up yourself :)

Set your monitor interface in this file: **/usr/local/zeek/etc/node.cfg **

<pre>[zeek]
type=standalone
host=localhost
interface=eth0
</pre>

Set your monitoring IP addresses here: **/usr/local/zeek/networks.cfg**. For example:

<pre>10.0.0.0/8 Private IP space
172.16.0.0/12 Private IP space 
192.168.0.0/16 Private IP space</pre>

Start Zeek cli:

<pre>zeekctl</pre>

Do the initial installation:

<pre>[ZeekControl] > install
[ZeekControl] > start
[ZeekControl] > deploy
[ZeekControl] > stop</pre>

To run Zeek on reboot, add the following to **/etc/rc.local** before exit 0:

<pre>ip link set eth0 promisc on
/usr/local/zeek/bin/zeekctl start

exit 0</pre>

And for some "scheduled maintenance":

<pre>crontab -e 
(select an editor and enter the following line)
 */5 * * * * /usr/local/zeek/bin/zeekctl</pre>

To send our logs into Splunk, we need the log to be json format, so edit **/opt/zeek/share/zeek/site/local.zeek** and add:

<pre># Output to JSON
@load policy/tuning/json-logs.zeek</pre>

Restart and confirm this works:

<pre>zeekctl deploy
tail /opt/zeek/logs/current/conn.log</pre>

## Splunk

We will be using Splunk to ingest our data so we can monitor and investigate logs. I used this because it seems to be one of the most popular tools for this scenario. You also have choises like the ELK stack to look into which is also very powerful for alerting. However, I'm going to use Splunk as it looks like there is currently more community support at the time of writing this.

![splunk logo](https://imgur.com/BUOJOEr.png)

You can choose to host this on whatever you like. I will be using my AWS student account. I've previously done this on a virtual machine and have it running all the time. I think a more realistic solution would be the cloud, it's reliable.

The installation is also versatile, you can deploy and launch a docker container on a virtual machine pretty quickly. 

[Splunk - Docker](https://hub.docker.com/u/splunk/#!)

Let's launch an EC2 instance (a virtual server hosted in the cloud). I chose Ubuntu Server 20.04 LTS. Choose the specifications to your needs.

Before launching, let's edit the security groups so it looks like the following: 

![AWS security group](https://imgur.com/2Jimxxp.png)

Port 8000 will be the web inteface for accessing splunk with our browser.

Setup a key, launch it and connect to it by SSH using whatever tool you want. I'm using a terminal emulator - **PuTTY**. You will need to find your key with Pageant or specify it by command line with "ssh ubuntu@ipaddress -i key". If you chose an Ubuntu server like myself, connect to the user "ubuntu".

I'll setup a new user called splunk:

<pre>sudo su
useradd -m -s /bin/bash splunk
passwd splunk</pre>

Change all user passwords while you're at it. Update and upgrade.

<pre>apt update && apt upgrade -y</pre>

Head over to the splunk website and get Splunk Enterprise, you will need to sign up if you haven't already.

[Splunk Downloads - Linux](https://www.splunk.com/en_us/download/splunk-enterprise.html#tabs/linux)

We want the **.deb** file. Let's install this with wget in our /opt directory. It should look something like this:

<pre>cd /opt
wget -O splunk-8.1.2-545206cc9f70-linux-2.6-amd64.deb 'https://www.splunk.com/bin/splunk/DownloadActivityServlet?architecture=x86_64&platform=linux&version=8.1.2&product=splunk&filename=splunk-8.1.2-545206cc9f70-linux-2.6-amd64.deb&wget=true'</pre>

Depackage it:

<pre>dpkg -i splunk-8.1.2-545206cc9f70-linux-2.6-amd64.deb</pre>

Enable splunk to start at boot, you'll need to accept the liscence agreement if this is your first time:

<pre>sudo /opt/splunk/bin/splunk enable boot-start</pre>

Start splunk:

<pre>systemctl start splunk
systemctl status splunk

● splunk.service - LSB: Start splunk
     Loaded: loaded (/etc/init.d/splunk; generated)
     Active: active (running) since Sat 2021-02-20 16:56:47 UTC; 2s ago
       Docs: man:systemd-sysv-generator(8)
    Process: 10472 ExecStart=/etc/init.d/splunk start (code=exited, status=0/SUCCESS)
      Tasks: 211 (limit: 1160)
     Memory: 369.3M
...</pre>

Cool, it's running. We can check the instance IP:8000 on our browser:

![Splunk Login](https://imgur.com/elPBqQ1.png)






