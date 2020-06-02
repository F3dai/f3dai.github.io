---
published: true
title: Networking Basics
category: Other
author: F3dai
image: 'https://i.imgur.com/FPcFw70.jpg'
---
A network is a collection of computers, servers, mainframes, network devices, peripherals, or other devices connected in order to allow for communication or the sharing of data. An example of a network is the internet which connects millions of devices across the world. 

**Networks are simply a way for machines / computers to communicate.**

At the physical level, it consists of all the machines you want to connect and the devices you use to connect them. Machines may be connected by a physical cable or wirelessly. To connect multiple machines together, each machine must connect to a hub or switch, and then those hubs / switches must connect together. In larger networks, each subnetwork is connected to the others by a router.

Here is a diagram of a simple network with different sub networks / subnets:

![network diagram](https://i.imgur.com/syOuasC.png)

This may be a small-sized home network or an office. There must be some connection points to the outside world. A barrier is set up between that network and the Internet, usually in the form of a firewall.

## Data packets

Data is sent when a connection is established across a network. The first part is to identify the destination host / where you want the data to be sent. All computers (as well as routers and switches), have an **IP address** that is a series of four numbers between 0 and 255 and is separated by periods, such as 192.168.0.1.

The data is then formatted for transmission. All data is [binary](https://en.wikipedia.org/wiki/Binary). This binary data is put into **packets**.

The first section of the packet is called the header, this is where information such as destination, source and how many packets they must except go. The majority of the time, there are 3 different headers - the IP header, the TCP header and the Ethernet header. 

## IP Addresses

Internet Protocol addresses are used for identifying network connected devices. Having an IP address allows for communication with other devices across a standard network like the internet. 

An **IP version 4 (IPv4)** address is a set of four three-digit numbers (for example 192.168.1.1) Each 3 digit number is within the range 0 - 255. The reason for this rule is that each binary octet cannot support a number larger than 255 (256 reserved).

![binary limit](https://i.imgur.com/h8CvEmR.png)

The first byte (first decimal number) indicates the network type / class. 

![IP classes](https://i.imgur.com/IeUtajH.png)

The IP address 127 (not listed in the table above) is reserved for the machine you are on. This is called the loopback address. This is another way of referring to the network interface card of the machine you are on.

It's also important to understand **private IP addresses**. In the early days of the internet, the mass number of networking devices we have today was not predicted. As a result, there were not enough IP addresses! Therefore, there are now internal IPs and external IPs. An internal / private IP is for identifying a device on a local network and an external IP is for identifying the network itself. 

Certain ranges of IP addresses have been designated for use within networks:

- 0.0.10 to 10.255.255.255 
- 16.0.0 to 172.31.255.255 
- 168.0.0 to 192.168.255.255

An example:

A computer may have a public IP of 77.11.22.44 but also have a private IP of 192.158.56.101. This means that data destined for this computer must go to 77.11.22.44, then traverse the internal network to find 192.158.56.101.

A gateway router performs what is called **network address translation (NAT)**. This takes the private IP on outgoing packets and replaces it with the public IP of the gateway router. 

**Subnetting** is simply splitting up a network into smaller portions. A subnet mask is a 32-bit number that is assigned to each host to divide the 32-bit binary IP address into network and node portions.

Example subnet mask: 255.255.255.0

A computer will take the network IP address as well as the subnet mask and use a binary [AND](https://www.electrical4u.com/and-operation-logical-and-operation/) operation to combine them.

- Class C network = 255.255.255.0
- Class B network = 255.255.0.0
- Class A network = 255.0.0.0

The decimal value 255 converts to 11111111 in binary, so you are therefore “masking” the portion of the network address that is used to define the network.

Subnetting allows you to use certain, limited subnets. Another approach is **classless interdomain routing (CIDR)**. 

Instead of the subnet mask, there will be an IP address followed by a slash and a number.

Example: 192.168.1.10/24

This method allows for [variable-length subnet masking (VLSM)](https://www.computernetworkingnotes.com/ccna-study-guide/vlsm-subnetting-explained-with-examples.html) that provides classless IP addresses. This is the most common way of defining network IP addresses. 

As I mentioned earlier, the traditional IPv4 address scheme is running out. IPv6 provides a solution to this. IPv6 is the successor to IPv4. IPv6 also introduces some new features to improve how the Internet works. 

Example IPv6 address: 2b0a:e1b:57:5d10:b50:c0a:2a8:102f

IPv6 utilizes a 128-bit address (instead of 32) and utilizes a hex numbering method. This gives 2128 possible addresses (many trillions of addresses). There is no subnetting, only CIDR.

Loopback address:  ::/128

**Uniform Resource Locator (URL)**

The main purpose of most people connecting to the internet is for accessing web pages. Thanks to **Domain Name Translation** , people do not have to type in IPv4 addresses into their browser to access a web page. Instead, people can input text (such as www.google.com or www.cybergoat.co.uk). A computer must translate the name you types in (Uniform Resource Locator) to an IP address. DNS handles this translation. If the correct domain name is entered, your computer will most likely send packets (using HTTP protocol) to TCP port 80. 

## MAC addresses

A **MAC address** is a unique address for a **network interface card (NIC)**. Every NIC in the world has a unique address that is represented by a six-byte hexadecimal number. 

The **Address Resolution Protocol (ARP)** is used to convert IP addresses to MAC addresses. 

Example MAC address: 00:25:96:FF:FE:12:34:56

The first 3 bytes are assigned to a vendor. This is also known as the Organizationally Unique Identifier (OUI). OUI helps determine the MAC address manufacturer. 

## Protocols

When networking, computers often require a language or rules across a network for successful communication. Different types of network communications are called **protocols**. 

Protocols allow for an agreed method of communication. There are many different protocols, each with their own purpose and usually operates on a specific port. Here are some examples of common protocols and their corresponding ports:

![common ports](https://i.imgur.com/2ozNjfX.png)

**Ports** are used to differentiate network traffic at each endpoint. If a computer is receiving packets, it must know whether it is an email or a file transfer. A port in networking terms is a handle, a connection point. 

You can read more about how ports are used to connect 2 different networking applications across a network on this article about socket programming:

[Socket Programming with Python (TCP and UDP)](/other/Socket-Programming/)