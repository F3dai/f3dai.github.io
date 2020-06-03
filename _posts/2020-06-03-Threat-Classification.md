---
published: true
title: Cyber Security Threat Classification
category: Ethical
author: F3dai
image: 'https://i.imgur.com/Fbe4jzs.png'
---
Networks constantly face security threats, and these can manifest themselves in a variety of forms. There are different ways we may choose to classify these threats to a system. You could choose to classify them by the **damage caused**, the level of **skill required** to execute the attack, or perhaps even by the **motivation** behind the attack. In this article, we will categorise attacks by what they do. Based on that philosophy most attacks can be identified as one of three generic classes:

- **Intrusion**
- **Blocking**
- **Malware**

![3 threats](https://i.imgur.com/XeKkpw3.png)

## Malware

Malware is the most commonly seen threat to any system, including home users' systems, smaller networks and even large enterprise networks. 

Malware is often designed to **spread by itself**. This means that the malware creator is not directly involved. As a result, malware is easily found across the internet, and more widespread. 

A computer virus is a very well-known example of malware. This is Nortons's definition of a computer virus:

[A computer virus, much like a flu virus, is designed to spread from host to host and has the ability to replicate itself. A computer virus is a type of malicious code or program written to alter the way a computer operates and is designed to spread from one computer to another. ](https://us.norton.com/internetsecurity-malware-what-is-a-computer-virus.html)

A virus essentially analogous to a biological virus in that both replicate and spread. The means of spreading can vary, for example, attackers often use victim e-mails to spread a virus to all contacts in the account.  

Viruses do not necessarily harm the system itself, but all of them cause network slowdowns or shutdowns as a result of heavy network traffic caused by the virus replication. 

Another type of malware is a **Trojan horse**. The name derives from an ancient tale, the city of Troy was besieged for a long period of time, but the attackers could not gain entrance. A large wooden horse was constructed as a "gift" for the city. This was taken into the protected area however the city didn't know that several soldiers were hidden inside the horse. The attacking soldiers inside the protected city opened the gates for the rest of the army to come inside. 

![trojan](https://i.imgur.com/2VycJF1.jpg)

This concept has been used to represent the digital Trojan horse as it illustrates the idea of disguising an item, appearing to be benign software but secretly downloading a virus or some other type of malware onto a computer. 

Trojan horses and viruses are the two most widely encountered forms of malware. The third category of malware is **spyware**. 

According to Malware bytes, spyware [is actually a type of malware that infects your PC or mobile device and gathers information about you, including the sites you visit, the things you download, your usernames and passwords, payment information, and the emails you send and receive.](https://www.malwarebytes.com/spyware/)

A **cookie** is a small piece of data sent from a website and stored on the user's computer by the user's web browser while the user is browsing. These are useful as they retain a lot of the website pages for faster load speed and remember actions. Cookies are often saved to a text file, meaning any data that the file saves can be retrieved by any website, so your entire Internet browsing history can be tracked.

**Key loggers** are another form of spyware. This type of spyware records all key strokes and sometimes take screenshots of the computer screen. 

## Intrusions

Attacks that attempt to gain access to the system are called **intrusions**. 

These attacks are aimed to intrude a specific system, also referred to as "hacking". This is attempting to access a system or network without permission, and usually with malicious intent. 

Attackers may use security flaws to leverage themselves into the system, but these types of attacks can be technologically much easier to execute. One example of this is **Social Engineering**, which relies more on human nature than technology. This is a broad term used to classify a type of malicious activity which is accomplished through human interactions. 

![Social Engineering](https://i.imgur.com/cW5XCCw.png)
[Source](https://smejoinup.com/what-is-social-engineering-attacks-how-to-prevent-an-attack/)

Social engineering uses techniques to get users to offer up the information needed to gain access to a target system. 

Here are 2 books that are widely recommended for understanding Social Engineering attacks:

[The Art of Deception: Controlling the Human Element of Security](https://www.amazon.co.uk/Art-Deception-Controlling-Element-Security/dp/076454280X/ref=sr_1_1?dchild=1&keywords=The+Art+of+Deception%3A+Controlling+the+Human+Element+of+Security&qid=1591176242&s=books&sr=1-1)

[Social Engineering: The Science of Human Hacking](https://www.amazon.co.uk/dp/111943338X?tag=duckduckgo-ffab-uk-21&linkCode=osi&th=1&psc=1)

The growing popularity of **wireless networks** offers attackers new methods of gaining access to a network. 

**War driving** is the act of searching for Wi-Fi wireless networks by an attacker usually in a moving vehicle, using a laptop or smartphone. People often forget that their wireless network signal often extends as much as 100 feet.

At DEFCON 2003, the annual hackers’ convention, contestants participated in a war-driving contest in which they drove around the city trying to locate as many vulnerable wireless networks as they could.

## Denial of Service

This is a **blocking attack**. In a Denial of Service attack (DoS), an attacker does not usually have access the system, but rather simply blocks access to the system from other / legitimate users. 

![DOS](https://i.imgur.com/1UEdcai.png)

According to Techopedia, [In a DoS attack, the attacker usually sends excessive messages asking the network or server to authenticate requests that have invalid return addresses. The network or server will not be able to find the return address of the attacker when sending the authentication approval, causing the server to wait before closing the connection.](https://www.techopedia.com/definition/24841/denial-of-service-attack-dos)

DoS attacks may also be used in conjunction with other types of attacks. For example, an attacker may use DoS so the network is unreachable by wireless means. An "**Evil Twin**" access point is employed to attract and trick people to enter credentials on the attackers fake AP. When an evil twin AP is present, a threat actor broadcasts the same SSID as the legitimate AP (and often the same BSSID or MAC address of the SSID) to fool the device into connecting.