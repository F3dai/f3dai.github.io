---
published: true
title: Cybersecurity Terminology
author: F3dai
category: Other
image: 'https://i.imgur.com/nEEOz9W.png'
---
Security professionals have specific terminology. These terms must be familiar with security or network professionals. Although most hacking terminology describes the activity or the person performing it (phreaking, sneaker, etc.). This article will discuss some of the basic security terminologies found in the cybersecurity industry.

## Firewalls and Proxy Servers

A firewall is a **barrier** between a network and the outside world. These are often software programs or hardware devices that **filter** and examine the information coming through your Internet connection. 

![firewall](https://i.imgur.com/nEEOz9W.png)
[Source](https://www.netstar.co.uk/why-do-i-need-a-firewall-business/)

These can exist as stand-alone servers, network routers or software running on a computer. 

They are designed to filter traffic entering and exiting a network.

Firewalls are commonly used in conjunction with a proxy server. A proxy is usually a computer which serves as a **hub** through which internet requests are processed. This hides your internal network IP address and presents another IP address as a public IP address.

Firewalls and proxy server are combined to provide a basic security perimeter. Incoming packets are **filtered**, but the internal network traffic is not affected. Sometimes there are intrusion-detection systems (IDS) implemented in these devices. 

## Access Control

Access control has the goal of **restricting resources** on the network. For exmaple, **logon procedures**, **encryption**, and any method that is designed to **prevent unauthorised personnel** from accessing a resource.

### Authentication

Authentication is derived from access control, perhaps the most basic security activity. This is the process of determining whether credentials or access (usernames / passwords) are authorised to access the network resource.

![authentication](https://i.imgur.com/vL86yzZ.png)
[Source](https://swoopnow.com/security-authentication-vs-authorization/)

Systems will attempt to authorise credentials, either granting or denying access. 

## Non-repudiation 

This term refers to the technique that is used to ensure that someone performing an action on a computer cannot falsely deny that they performed that action. 

Nonrepudiation is a method of **guaranteeing message transmission** between parties via digital signature and/or encryption. 

This is particularly important on networks as it can provide **reliable records** of what user took a particular action at a specific time. This is a way of **tracking** the actions taken by a user. 

System administrators use **logs** as a form of Non-repudiation. This article describes is an example of how System Log files are analysed for cybersecurity:

[Analysing Windows Log Files](/other/Analysing-Windows-Logs/)

## Least Privilege

When system administrators assign privileges to users, they will find themselves using the term Least Privilege. The **principle of least privilege (PoLP)** has the idea that users must be able to access only the information and resources that are necessary for its legitimate purpose. 

This is assigning the **minimum privileges** required for that person to do his job.

This diagram represents the system of privileges on an operating system (privilege rings for the Intel x86).

![kernel](https://upload.wikimedia.org/wikipedia/commons/thumb/2/2f/Priv_rings.svg/1280px-Priv_rings.svg.png)
[Source](https://en.wikipedia.org/wiki/Principle_of_least_privilege)

An effective example of Least Privilege is my computer - I strictly operate on a non-administrative account so **when** there is an attack on my system, I can **control and minimise** the damage caused when there is malicious activity is on my computer. 

## Hacking Terminology:

![BWG hats](https://i.imgur.com/PmmLfBA.png)

### White hat hackers

White hat hackers are perceived as the ethical category of "hackers". They will usually **find and report** vulnerabilities on a system. 

These types of people choose to use their powers for good rather than evil.

They employ similar methods of hacking as black hats, with one exception- they do it with **permission** from the owner of the system first, making the process completely **legal**. 

### Black hat hackers 

These types of hackers are most commonly depicted in the media. Their intentions are **malicious**, with the goal of causing harm to a system. These types of people are regarded as **unethical**.

Black hat hackers often steal data, erase files, or deface websites. These guys may be responsible for writing malware.

The intentions of black hack hackers can vary. The motivation may be for personal or **financial gain**, **cyber espionage**, **protest** or perhaps are just addicted to the **thrill** of cybercrime

### Grey hat hackers 

People who typically law-abiding citizens, but in some cases will venture into illegal activities are referred to as grey hat hackers.

Motivations can widely vary - conducting illegal activities for reasons they feel are ethical, for example, accessing a system belonging to a corporation that the hacker feels is engaged in unethical activities. 

Grey hat hackers may also look for vulnerabilities in a system without the owner’s permission or knowledge. If issues are found, they will report them to the owner, sometimes requesting a small fee to fix the issue.
