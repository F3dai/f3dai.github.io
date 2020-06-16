---
published: false
---
Attacktive Directory - "99% of Corporate networks run off of AD. But can you exploit a vulnerable Domain Controller?" This is a TryHackMe box. To access this you must sign up to [https://tryhackme.com/](https://tryhackme.com/).

**URL:** [Year of the Rabbit](https://www.tryhackme.com/room/yearoftherabbit)

**Difficulty:** Medium

**Author:** Sq00ky

## Enumeration

We are given the IP 10.10.62.162. Run an nmap scan with the following command:

<pre>nmap -p- -A -o portscan 10.10.113.38</pre>

Here are the open ports:

<pre></pre>

Let's visit port 80:

![port 80](https://imgur.com/vsNyWnX.png)

The image on this page is a link to the following website:

[https://www.iis.net/?utm_medium=iis-deployment](https://www.iis.net/?utm_medium=iis-deployment)

We already know this box will be based around Internet Information Services (IIS). 

<pre>PORT      STATE SERVICE       VERSION
53/tcp    open  domain?
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|_    bind
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2020-06-16 15:45:01Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: spookysec.local0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: spookysec.local0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: THM-AD
|   NetBIOS_Domain_Name: THM-AD
|   NetBIOS_Computer_Name: ATTACKTIVEDIREC
|   DNS_Domain_Name: spookysec.local
|   DNS_Computer_Name: AttacktiveDirectory.spookysec.local
|   Product_Version: 10.0.17763
|_  System_Time: 2020-06-16T15:47:30+00:00
| ssl-cert: Subject: commonName=AttacktiveDirectory.spookysec.local
| Not valid before: 2020-04-03T18:40:09
|_Not valid after:  2020-10-03T18:40:09
|_ssl-date: 2020-06-16T15:47:44+00:00; +2s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49674/tcp open  msrpc         Microsoft Windows RPC
49675/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49676/tcp open  msrpc         Microsoft Windows RPC
49679/tcp open  msrpc         Microsoft Windows RPC
49688/tcp open  msrpc         Microsoft Windows RPC
49696/tcp open  msrpc         Microsoft Windows RPC
49793/tcp open  msrpc         Microsoft Windows RPC</pre>

A lot of ports open which seem to all point towards an Active Directory environment. 

We can see **Kerberos** running on port 88 which is an authentication protocol. This is currently the default authentication technology used by Microsoft to authenticate users to services within a local area network.

Port 445 is also open which indicates a service called the server message block (SMB) over TCP/IP.

Whenever ports 139 and 445 are open, I always start by using **enum4linux** to enumerate.

<pre>enum4linux -A 10.10.113.38</pre>

![enum4linux](https://imgur.com/MbN6yLQ.png)

We've got some interesting information back. We've identified some known usernames:

<pre>Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none</pre>

And also a domain name:

<pre>Domain Name: THM-AD</pre>

I will be installing a tool called Kerbrute. This essentially bruteforces and enumerates valid Active Directory accounts through Kerberos Pre-Authentication.

[Kerbrute - Github](https://github.com/ropnop/kerbrute)

Let's install this tool on the local machine with [golang](https://www.ostechnix.com/install-go-language-linux/)






