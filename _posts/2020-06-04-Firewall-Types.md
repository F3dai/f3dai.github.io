---
published: true
title: Types of System Firewalls
author: F3dai
image: 'https://i.imgur.com/YWEJF8h.png'
category: Other
---
Several types of firewalls offer their advantages and disadvantages. As a system administrator, it's vital to understand each of the firewalls and their benefits.

Here are a few of the the basic types of firewalls:

- Packet filtering
- Application gateway
- Circuit level gateway
- Stateful packet inspection

## Packet filtering

This is a firewall technique used to control **network access** by monitoring packets going into and out of the network and allowing or denying them based on the **source** and **destination** Internet Protocol (IP) addresses, **protocols** and **ports**.

Ths the most basic type of firewall, also known as **screening firewalls**. Only packets that match the firewalls security criteria are allowed through. 

Many common OS such as Windows and Linux operate on a basic packet filtering software. 

![packet filter](https://i.imgur.com/TQpsVH3.png)
[Source](https://docstore.mik.ua/orelly/networking_2ndEd/fire/ch05_02.htm)

These are some of the parameters the firewall will use when examining a packet:

- Source address 
- Destination address
- Source port
- Destination port
- Protocol type

These firewalls are very **easy to configure** and **inexpensive**.

These types of firewalls are susceptible to either a ping flood or SYN flood, as they do not actually examine the packet contents or compare them to previous ones. There is no authentication, as it only analyses the packet header.

Here are some considerations when configurating the firewall:

- What source ports to allow
- What source IP addresses to allow
- What destination ports to allow
- What types of protocols to allow (FTP, SMTP, POP3, etc.)

## Stateful Packet Inspection

This technology ensures all inbound packets are as a result of outbound requests. This was designed to prevent malicious packets from entering a network or computer. This is an improvement on basic packet filtering.

Each packet is **examined**, granting or denying access to the network or computer. It also takes into consideration the **previous packets**. As a result, the firewall is aware of the sessions and **context** of every packet. 

This means that Stateful Packet Inspection will be able to understand wether the packet is part of a unusually large stream of packets, indicating a DoS attack. 

They can also identify **IP spoofing**, by examining the source IP that appears to come from the inside of the firewall. This provides the capability of advanced filtering.

This is the recommended type of firewall for most systems - Home routers will have this technology enabled. 

## Application Gateway

This is a program that runs on a firewall. This **negotiates** (process of authentication and verification) with multiple types of applications to allow their traffic to pass the firewall. 

In this process, an application gateway will **analyse** the client application and the server-side application. 

![application](https://i.imgur.com/YWEJF8h.png)
[Source](https://docs.microsoft.com/en-us/azure/application-gateway/media/application-gateway-create-gateway/scenario.png)

It will then decide if the client application’s traffic is permitted through the firewall. This enables network administrators to choose which services are allowed on the network, such as FTP, Telnet, Web etc.

When a session is established, the Application Gateway will act as a **proxy**, which the client negotiates with. This means that the process creates two connections - to and from the proxy. 

Although this type of firewall is effective, it uses a lot of **system resources** such as a high CPU time. Due to the time taken, it can be susceptible to DoS attacks. A flood of connection requests may overwhelm the firewall. 

## Circuit Level Gateway

This type of firewall will provide User Datagram Protocol (UDP) and Transmission Control Protocol (TCP) connection security. These are generally implemented on **high-end equipment**. 

**User authentication** is employed at the start of the process.

Firstly, the application is checked for allowing or denying access, then the user is authenticated. This authentication is the first step. The credentials are checked, before allowing access to the user. 

This allows for **individual verification** by username or IP address before any further communication is permitted. 

A **virtual “circuit”** now exists between the internal client and the proxy server.

Requests to the internet will be processed by this proxy, changing the IP address of the request. External users will only identify the proxy IP address. The responses are then reprocessed by the proxy, routing back to the client. 

This "virtual client" is what makes this system secure. 
