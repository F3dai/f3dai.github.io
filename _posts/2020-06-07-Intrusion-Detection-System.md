---
published: True
title: Intrusion Detection Systems - Concepts and Examples
category: Other
author: F3dai
image: https://imgur.com/Hk9oY43.png
---
There are six basic approaches to intrusion-detection and prevention. Some of these methods are implemented in various software packages, and others are simply strategies that an organisation can employ to decrease the likelihood of a successful intrusion.

## Concept

Let's first look back when [packet sniffing](https://www.techopedia.com/definition/4113/sniffer) first arose. Back when **hubs** were used instead of switches, a hub would direct a packet to a local machine by the MAC address. It would do this by sending it to all the machines and only the matching MAC address would accept - the rest would ignore. People soon realised that choosing to not ignore the packet would allow them to see all the packets on the network. This was one of the indications of the introduction to **Intrusion Detection Systems**.

### Pre-emptive blocking

Pre-emptive blocking seeks to prevent intrusions before they occur

- Observe** any imminent threats, then block the origin IP address.
- Can be quite complicated as there are many false positives(blocking legitimate users)
- Distinguishing malicious and legitimate traffic is the difficult part.
- This sort of approach should only be one part of an overall intrusion-detection strategy and not the entire strategy.

### Anomaly Detection

This concept introduces actual software that works to detect intrusion attempts and notify the administrator. 

- System will simply look for abnormal behaviour.
- Activity that does not match normal users is considered abnormal.
- Profiles are usually kept about users, groups or applications.
- “Trace back” detection or process.

Here are some of the ways in which an anomaly is detected:

- Threshold monitoring: Pre-sets acceptable behaviour levels, and observing any exceeding levels (Log in attempts, connections etc).
- Resource profiling: Measures system-wide use of resources and develops a historic usage profile.
- User/group work profiling: Individual user / group profiles. Profiles are updated with activities, while the system monitors short and long-term profiles.
- Executable profiling: Measure and monitor the resources used by programs. This system focuses on programs that can not be tracked back to a user.

![IDS placement](https://imgur.com/PtYV2MQ.png)

### Terminology:

An activity: an interesting data to the operator.
Administrator: Person responsible for organisational security.
Sensor: IDS component which collects data.
Analyser: IDS component which analyses the collected data.
An alert: Message from analyser, indicating an interesting event.
Manager: The part used to manage. eg: console.
Notification: When the IDS makes the operator aware of an alert, usually on the manager.
Operator: Usually the administratory - the person responsible for the IDS.
Event: Indication of potentially suspicious activity.
Data source: Information used by the IDS to detect suspicious activity.

An active IDS, now called an Intrusion Prevention System (IPS).

A passive IDS logs activities and sometimes alerts the administrator. 

A host-based intrusion-detection system (HIDS) monitors just a single machine or host-based intrusion prevention system (HIPS).

A network-based intrusion prevention system (NIPS) detects traffic on a network segment, rather than a single machine.

Here is an illustration of the differences between an IDS and IPS:

![IPS IDS](https://imgur.com/GY81i6V.png)
[Source](https://www.youtube.com/watch?v=cGIgJOICpX0)

## Common IDS systems:

### Snort

[www.snort.org](https://www.snort.org/)

![snort](https://imgur.com/Hk9oY43.png)

This is probably the most well-known open-source IDS available today. This software is implemented on a server to monitor incoming traffic.

Snort works best on a host-based firewall in a system where both Snort and the firewall run on the same machine. 

Snort works in one of three modes: 

- Sniffer: Display continuous stream of packets on a console.
- Packet logger: Similar to Sniffer mode, but everything is logged to a file.
- Network intrusion-detection: Snort uses a heuristic approach to detecting anomalous traffic. This mode is rule-based and it learns from experience.

### Cisco Intrusion Detection and Prevention

[Next-Generation Intrusion Prevention System (NGIPS)](https://www.cisco.com/c/en/us/products/security/ngips/index.html)

Cisco advertises some of the following features on their IDS systems:

- Visibility
- Efficacy (rules and signatures every two hours)
- Operational cost (increase operational efficiency and reduced overhead)
- Flexibility (flexible deployment options)
- Integration (plugs into your network without major hardware changes)
- High-performance appliances (4100 Series and 9000 Series)

![products](https://imgur.com/xOxKi1t.png)

** Please note, these systems are no longer being sold **

Cisco provided two widely used IDS products: 

- The Cisco IDS 4200 Series Sensors.
- Cisco Catalyst 6500 Series Intrusion-Detection System (IDSM-2) Services Module.

The products include malware protection as well as sandboxing.

The 4100 series is meant for small networks and the 9000 series is designed for large scale networks.


