---
published: true
title: Honeypots - Intrusion Detection Concepts
category: Other
author: F3dai
image: 'https://imgur.com/hQ62l5e.png'
---
Creating an **intentionally vulnerable system** to attract adversaries is called a honeypot. As the name suggests, system administrators create an enticing item, such as a valuable server or even an entire subnetwork, to focus the attacker on the honey pot rather than the rest of the system. 

This article will be outlining the concepts of a honeypot with an example.

## Concept

Software can closely monitor everything that happens on the intentionally vulnerable or "valuable" system, enabling tracking and perhaps identification of the intruder.

The underlying concept of a honeypot means that any traffic detected on that machine or network is considered suspicious. 

The honeypot is not an actual machine, it is created for only attackers to connect to. These are sometimes designed to entice an attacker to remain for a long period, allowing the organisation's security team to trace where the attacker came from.  

Below is a diagram which illustrates how system administrators can set up a honey pot within their domain:

![honeypot position](https://imgur.com/I5tKwXd.png)

## Specter IDS

**SPECTER** is a smart honeypot-based intrusion detection system. Systems that run specter will have a dedicated machine which emulates the major Internet protocols/services such as HTTP, FTP, POP3, SMTP, and more. Specter aims to create an interesting target to lure hackers away from the production machines. 

The software closely analyses and logs all incoming traffic. There are 5 different modes the software can operate in:

- Open: Behaves like a badly configured server. This primarily attacks lower-skilled hackers.
- Secure: Behaves like a secure server.
- Failing: System behaves like a server with hardware and software problems. This is attractive as the system is likely to be vulnerable.
- Strange: Behaves in unpredictable ways. This mode attracts more talented hackers and entices them to stay on longer to understand the system. 
- Aggressive: Actively tries to identify and trace the attacker. 

Specter attempts to leave a trace on the attacker's machine, creating evidence for any criminal activity.  

## Symantec Decoy Server

Symantec is a well-known vendor for antivirus software and firewall solutions. They also provide a honeypot product which initially started as a decoy server which simulated e-mail traffic.  

The decoy server works as an IDS and honeypot which monitors any potential intrusions. If anything is detected, the decoy server will record all related traffic for further investigation. 

### Intrusion Detection Systems:

[www.cybergoat.co.uk/other/Intrusion-Detection-System/](/other/Intrusion-Detection-System/)
