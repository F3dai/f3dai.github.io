---
published: true
title: Pickle Rick - TryHackMe Walkthrough
category: Writeup
image: 'https://imgur.com/Bk9cyE7.png'
author: F3dai
---
This Rick and Morty themed challenge requires you to exploit a webserver to find 3 ingredients that will help Rick make his potion to transform himself back into a human from a pickle. This is a TryHackMe box. To access this you must sign up to [https://tryhackme.com/](https://tryhackme.com/).

**URL:** [tomghost](https://www.tryhackme.com/room/picklerick)

**Difficulty:** Easy

**Author:** tryhackme

### Enumeration

We are given the IP 10.10.213.134. Run an nmap scan with the following command:

<pre>nmap -p- -A -o portscan 10.10.213.134</pre>

These are the open ports:

<pre>PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 10:98:cf:44:dd:60:35:1b:00:47:61:3a:15:80:2c:ad (RSA)
|   256 67:2c:e7:5e:fc:b3:72:98:76:a6:cf:59:b7:d5:e0:f0 (ECDSA)
|_  256 83:8f:f6:cf:cf:af:a3:cd:bb:c9:61:e3:df:0c:41:d0 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Rick is sup4r cool
</pre>

![nmap](https://imgur.com/qbYR2v2.png)

The most common ports are open - 22 and 80. Let's check out port 80 on my browser:

![port 80](https://imgur.com/jBK3Fmk.png)

Doesn't seem to be much, just references to the TV Show Rick and Morty. However, I noticed a comment in the page source: 

![source](https://imgur.com/jpKzLz2.png)

<pre> &lt;!--

Note to self, remember username!

Username: R1ckRul3s

--&gt;</pre>
  
We have a username R1ckRul3s. 
  
Let's run a dirb scan to find any hidden paths:
  
<pre>dirb http://10.10.213.134</pre>
  
![dirb](https://imgur.com/RlYQOqp.png)

Not much has been revealed, a robots page, assets and server-status. 
  
Robots.txt has "Wubbalubbadubdub":
  
![robots](https://imgur.com/4Z9t9OM.png)
  
This doesn't seem like a hidden path. I tried using **R1ckRul3s:Wubbalubbadubdub** for ssh but it didn't work. I will use nikito to enumerate further. 

<pre>nikto -h 10.10.213.134</pre>

![nikto]()

We found a login page:

<pre>http://10.10.213.134/login.php</pre>

![login](https://imgur.com/bDYc7EH.png)

Using the credentials here gives us authenticated access:

![login](https://imgur.com/IIcHdub.png)

## Ingredient 1

The landing page for Ricks Portal has command execution. I executed the shell command "ls" to list the current directory:

![ls](https://imgur.com/iPiYZcd.png)

We haven't viewed Sup3rS3cretPickl3Ingred.txt, let's open this up in our browser:

![ing](https://imgur.com/Gh3nUsm.png)

<pre>mr. meeseek hair</pre>

This looks like our first Ingredient or flag for this CTF challenge. 

## Ingredient 2

I traverse to the home directory to see other users that are on the system. There is "ubuntu" and "rick". Remember, command inject means you cant cd, you must use ls to explore the system.

<pre>ls -la /home/rick</pre>

![ls rick](https://imgur.com/OO8ktEY.png)

The command "cat" is disabled on this system so we should use less to see the contents of this file.

<pre>less /home/rick/'second ingredients'</pre>

![less](https://imgur.com/1ijHXsX.png)

<pre>1 jerry tear</pre>

Now we're onto our third Ingredient. 

## Ingredient 3

I explored a lot of the file system and couldn't find much else. The only section I haven't gone into is the root directory, however, I do not have sudo rights. Executing the following command returns nothing:

<pre>ls -la /root</pre>

This command shows the sudo rights of the current user:

<pre>sudo -l</pre>

![sudo l](https://imgur.com/jilTp35.png)

<pre>
User www-data may run the following commands on ip-10-10-213-134.eu-west-1.compute.internal:
    (ALL) NOPASSWD: ALL</pre>

This means the current user can execute anything! Usually, I would execute sudo /bin/sh to get a root shell but since this is a limited command injection, we can just use sudo before our command.

<pre>sudo ls -la /root</pre>

![root](https://imgur.com/XTb6Nwu.png)

<pre>sudo less /root/3rd.txt</pre>

![3rd](https://imgur.com/atQbaxz.png)

We have the last Ingredient: 

![fleeb](fleeb juice)

<pre>fleeb juice</pre>

![potion](https://imgur.com/WLzYZkY.png)



