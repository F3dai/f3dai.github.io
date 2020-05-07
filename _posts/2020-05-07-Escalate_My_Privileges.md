---
published: false
---
URL: [Escalate Privs Vulnhub](https://www.vulnhub.com/entry/escalate-my-privileges-1,448/)

Difficulty: Easy / Beginner Level

Author: Akanksha Sachin Verma

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

