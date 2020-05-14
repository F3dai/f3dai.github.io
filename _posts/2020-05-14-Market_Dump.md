---
published: true
category: HackTheBox
title: Market Dump
---

## Market Dump

This is a Hack The Box forensics challenge writeup.

![htb](https://imgur.com/zjlHjW4.png)

**Description:** We have got informed that a hacker managed to get into our internal network after pivoiting through the web platform that runs in public internet. He managed to bypass our small product stocks logging platform and then he got our costumer database file. We believe that only one of our costumers was targeted. Can you find out who the customer was? 

## Method

We have been given just one file:

<pre>MarketDump.pcapng</pre>

If you come across a file which is unfamiliar to you, the command "file" can be extremely useful. The "file" command is used to determine the type of a file.

<pre>file MarketDump.pcapng</pre>

![file](https://imgur.com/n1HLeD0.png)

.pcap files are usually associated with Wireshark, a tool used to analyse data files that have recorded network traffic. Read more about this [here](https://www.reviversoft.com/file-extensions/pcap).

Let's see what we can find when we open the file with Wireshark. I will be using pre-installed Wireshark on Kali Linux.

Here is a snippet of some traffic:

![sniff](https://imgur.com/dtmDkYE.png)

Each entry in Wireshark represents a packet that has been captured on the network. The green packets seen on this screenshot are requests for web-pages, we can tell from "HTTP" in the protocols column. 

This challange description mentioned a hacker that gained access to the network through the web platform so we need to look further down the traffic to find something.

Further on we see a stream of telnet traffic. Telnet is essentially a protocol used to communicate with remote hosts the same way ssh does but without the same security. 

Here is one of the packets sent near the start of the stream:

![tel](https://imgur.com/M7vVMjB.png)

It seems like a poor grammar message to the possible hacker. "Here is you're daily stock report". 

![ls](https://imgur.com/j1liUUB.png)

Here is another packet where the user executed the command "ls".

There is a lot of data to go through. When there are a lot of packets in .pcap files and I am trying to find a string, I use the command "strings" to filter out all the data. This command essentially prints out all the readable ASCII characters in a file. 
[Here](https://www.howtogeek.com/427805/how-to-use-the-strings-command-on-linux/) is a good article on this. 

The pcap file has a lot of data so we will output the results to a file to analyse.

<pre>strings MarketDump.pcapng > stringsdump</pre>

After analysing all the strings from the .pcap file, we can see all the different activities presented as strings. Mostly American Express digits.

One of the strings that caught my eye was this:

![american](https://imgur.com/KHSOoHv.png)

This stands out, and it looks like encrypted text.

<pre>NVCijF7n6peM7a7yLYPZrPgHmWUHi97LCAzXxSEUraKme</pre>

This following website is a great tool for identifying ciphers:

[Cyber Chef](https://gchq.github.io/CyberChef/)

Use the "magic" option and it should give the correct result in plaintext.

![Cyber Chef](https://imgur.com/p4RsM4m.png)


And the flag has been found.

<pre>HTB{DonTRuNAsRoOt!MESsEdUpMarket}</pre>
