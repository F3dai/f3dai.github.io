---
published: true
title: 'Basic Network Utlities (ipconfig, ping, tracert, netstat)'
image: 'https://i.imgur.com/bN8P71n.png'
category: Cheatsheet
author: F3dai
---
This article will explain the different basic network utilities. You can execute some network utilities from a command prompt (Windows) or from a shell (Unix/Linux).

I will be using screenshots from a Windows command-prompt perspective. 

## Ipconfig

Windows:

<pre>ipconfig</pre>

Linux:

<pre>ifconfig</pre>

One of the most important commands on a networking device. This command is executed to display information about your own system. 

Go to Start > Run > type "cmd" to get a command prompt.

![how to ipconfig](https://i.imgur.com/W9SVMJ8.png) 

You could input the same command in UNIX or Linux by typing in ifconfig from the shell. This is an example of the results you can expect:

![ipconfig pshell](https://i.imgur.com/6uhBQ4z.png)

This command gives you information about your connection to a network (or to the Internet). Most importantly, you find out your own **IP address**. You can also find your **default gateway** (connection to the outside world).

Some additional parameters are also important. You might use several options to find out different details about your computer’s configuration. The following command is commonly used to find a more extensive list of network configuration.

<pre>ipconfig/all </pre>

![ip all](https://i.imgur.com/mkEZh9b.png)

## Ping

Windows + Linux:

<pre>ping [ip address]</pre>

Ping is a computer network administration software utility which is used to send a test packet, or echo packet, to a machine to find out whether the machine is reachable and how long the packet takes to reach the machine.

![ping](https://i.imgur.com/8FHsAXx.png)

A 32-byte echo packet was sent to the destination and returned. TTL means Time To Live. The time unit (Time = 74ms) is the length of time the packet should take to the destination before giving up. 

## Tracert

Windows:

<pre>tracert [ip address]</pre>

Linux:

<pre>traceroute [ip address]</pre> 

This command is an advanced / "deluxe" version of ping. Tracert not only tells you whether the packet got there and how long it took, but it also tells you all the intermediate hops it took to get there. 

![tracert](https://i.imgur.com/sDSliNE.png)

As the internet is such as large network, packets always hop across several routers. This command shows which nodes the packet is sent to get to the destination. In this case, I have followed the steps to get to www.cybergoat.co.uk.

## Netstat

Windows + Linux:

<pre>netstat</pre>

Netstat is an abbreviation for Network Status. This useful command essentially tells you what connections your computer currently has. There will most likely be numerous private IP addresses (such as 192...) 

![netstat](https://i.imgur.com/wSgGkMF.png)

These four commands we have just examined are the **core utilities**. These four (ipconfig, ping, tracert, and netstat) are absolutely essential to any network administrator.
