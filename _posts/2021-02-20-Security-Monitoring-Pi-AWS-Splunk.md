---
published: true
title: Pi + IDS = Security Monitoring
author: f3dai
image: 'https://imgur.com/55RpuAS.png'
category: Other
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

Not going to cover booting up the OS for the first time. Setup headless / connect to monitor, whatever floats your boat. Find the IP through your router interface if headless.

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

![AWS security group](https://imgur.com/4jfzef9.png)

Port 8000 will be the web inteface for accessing splunk with our browser and port 9997 will be how Splunk Enterprise will receive logs. 9997 is the standard port to use but you can use any others, I will get into this later on and you can always change it later.

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

Log in with the credentials you specified. Now that this is up, we can look at setting up how we send logs into our Splunk instance.

## Forward Data to Splunk Enterprise

We will be using Splunk's Universal Forwarder to ship our logs to Splunk. The most common way to use the universal forwarder is to send data to a Splunk Enterprise indexer or indexer cluster. You can have multiple forwarders sending data into your Splunk indexers. See this topology provided by Splunk:

![Forwarder Topology](https://imgur.com/ivn5OF2.png)

_**We will be installing the UF on our Pi as this will be collecting our logs we want to ship to Splunk**_

**Let's configure receiving on Splunk Enterprise first**:

Enable a receiver. The receiver will be considered an indexer or a cluster of indexers. Go to Settings > Forwarding and receiving. 

![Receiving Splunk](https://imgur.com/NKK3KCz.png)

Create a new receiving port, the conventional port is 9997 but you can choose whatever you like. Make sure this port has been whitelisted / added to your security group in the AWS console. 

![Receiving Port](https://imgur.com/bNfP4nt.png)

Save. 

**Installing the Universal Forwarder:**

[Universal Forwarder - Downloads](https://www.splunk.com/en_us/download/universal-forwarder.html)

Choose Linux, whichever distribution based on the OS of your Pi. Install to _/opt/splunkforwarder_. My wget command:

<pre>wget -O splunkforwarder-8.1.2-545206cc9f70-linux-2.6-amd64.deb 'https://www.splunk.com/bin/splunk/DownloadActivityServlet?architecture=x86_64&platform=linux&version=8.1.2&product=universalforwarder&filename=splunkforwarder-8.1.2-545206cc9f70-linux-2.6-amd64.deb&wget=true'
dpkg -i splunk_package_name.deb</pre>

You may choose to add /opt/splunkforwarder/bin to your PATH in ~/.profile, the same way we did earlier on in this article. So $SPLUNK_HOME in this article will be wherever you installed the UF. /opt/splunkforwarder in this case.

**Start the UF:**

<pre>$SPLUNK_HOME/bin/splunk start --accept-license</pre>

And restart whenever making a config change:

<pre>$SPLUNK_HOME/bin/splunk stop</pre>

Pretty straight forward. So let's make some config changes!

These are the important config files you should be aware of:

- **inputs.conf** controls how the forwarder collects data.
- **outputs.conf** controls how the forwarder sends data to an indexer or other forwarder.
- **server.conf** for connection and performance tuning.
- **deploymentclient.conf** for connecting to a deployment server.

So let's specify the data forwarding at _$SPLUNKHOME/etc/system/local/outputs.conf_. This is mine:

<pre>[tcpout]
defaultGroup = default-autolb-group

[tcpout:default-autolb-group]
server = receiving-instance-ip:9997

[tcpout-server://receiving-instance-ip:9997]</pre>

Reload the UF and we should expect the events to go into our Splunk Server:

<pre>$SPLUNKHOME/bin/splunk restart</pre>

We can now optionally install an app for Splunk which will help parse and create dashboards for ourselves. You have 2 options:

[Corelight - featuring dashboards](https://splunkbase.splunk.com/app/3884/)

[Splunk Add-on for Zeek aka Bro - no dashboards, Splunk Built](https://splunkbase.splunk.com/app/1617/)

I used the Splunk Built option, it's up to you. Install the app, go to **Splunk > Apps > Manage Apps > Install from file > reboot**. 

Then go to **Settings -> Knowledge -> Event types > App dropdown > Corelight App and select Corelight_idx on the page**:

![Corelight](https://imgur.com/dNDfoyn.png)

Then make the search string _index=zeek_ like so:

![search string](https://imgur.com/aTX99WH.png)

I found this log is good for debugging:

<pre>tail var/log/splunk/splunkd.log</pre>

You should see something like:

<pre>INFO  TcpOutputProc - Found currently active indexer. Connected to idx=instance-ip:9997, reuse=1.</pre>

Check if you can see logs by using the search query:

**index=zeek**

## SPAN (Switched Port Analyzer)

This part is optional, and only relevant if you're using a switch.

If you are going for my first recommended tology with a switch, you'll need to port mirror to configure the SPAN port. I'm using a GS305E, Netgear switch. Regardless of the make it should be pretty straight forward to setup port mirroring. This is what I have plugged in by ethernet:

- Port 1: Router
- Port 2: RPi (IDS)
- Port 3: Ethernet Device 

![port mirroring](https://imgur.com/FCTGXyn.png)

If you're using your own wireless AP, make sure you attach this to the switch and mirror it to port 2 or whatever your IDS is on.

You may as well just mirror all of your ports to your IDS, tho nothing is plugged in. Future you can just plug into the switch without logging onto the interface to port mirror.

## Pi AP

This part is optional, and only relevant if your Pi is capable of becoming a wireless AP. 

I found this github repo very useful and effective. It does the job. However, it's no longer maintained so this may only be relevant at the time of writing this. 

[create_ap - github](https://github.com/oblique/create_ap)

I imagine there are lots of other lovely github repos, or you can just do it manually:

[RPi Bridged Wireless AP](https://www.raspberrypi.org/documentation/configuration/wireless/access-point-bridged.md)

Again, this part varies depending on how you want your topology but I didn't want to create any subnets so I just created a bridged AP. 

## fin

Any device you want monitored, you attach to your switch by ethernet or connect to your Pi AP. 

Thanks!





