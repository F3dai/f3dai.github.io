---
published: true
title: VPN Concepts and Protocols
image: 'https://imgur.com/9mPYaYu.png'
category: Other
author: F3dai
---

Virtual Private Networks (VPNs) are commonly used to **remotely and securely** connect to a network. A private network is created over the internet in order to connect hosts. Security is accomplished by creating a **cryptographic tunnel**. 

## VPN Concepts

![tunnel](https://imgur.com/bc9aXm0.png)

Remote users can securely access a network as if it was on the **local network**. Organisations may need remote employees accessing the network resources, creating a security concern.

A VPN emulates a direct network connection. In other words, the VPN provides both the same level of access and the same level of security as a direct connection.

Data must be encapsulated or wrapped with a header which contains routing information to transmit the information from one point to another across the internet. As a result, a virtual network connection is created between the two points. The data is encrypted, thus making that virtual network private. No separate physical technology is needed, VPNs use existing connections.  

![tunnel vision baby](https://imgur.com/9mPYaYu.png)

VPNs provides **encryption** and **authentication**. 

VPN technology is also useful for organisations that need a site-to-site connection. A VPN would enable this connectivity with multiple links. This could avoid expensive, dedicated lines and simply use existing Internet connections.  

## Protocols

VPN protocols are frequently used to allow for encryption. Point-to-point Tunneling Protocol (PPTP) and Layer 2 Tunnelling Protocol (L2TP) are the two most commonly used protocols. The part of the virtual connection where data is encapsulated is called the **tunnel**. The combination of L2TP  and IPSec often achieves a higher level of security.

### PPTP:

This tunnelling protocol allows PPP (point-to-point protocol, an older protocol) to have its packets **encapsulated within the IP packets** and forwarded through any IP network (like the internet). This is considered more secure than PPP 

PPTP is used to create VPNs. This may be considered **less secure than L2TP**, but it uses fewer resources and is supposed by almost every VPN implementation.  

This technology is still widely used despite newer protocols. It operates on the **data link layer** (layer 2 of OSI model), allowing different networking protocols to operate on this PPTP tunnel. **Authentication** is also important when connecting remote users, therefore, PPTP allows 2 separate technologies: **Extensible Authentication Protocol** (EAP) and **Challange Handshake Authentication Protocol** (CHAP) 

- EAP was designed for PPTP, and works with PPP. It works within PPP's authentication protocol. Multiple authentication methods may be used. 

- CHAP uses a 3-way handshake process for authentication. Once the connection is established, a challenge message is sent to the client. A hash value (one-way calculation) is sent, where the server checks the response against its calculation of the expected hash. Authentication is accepted if they match. This process is periodically repeated. 

![3 way shake](https://imgur.com/xj4fTYy.png)

### L2TP:

This extension of Point-to-Point (PPP) is often used for virtual private networks across the internet. It is regarded as a new and improved version of PPTP. It operates on the data link layer (Layer 2 of OSI). PPTP and L2TP are often considered less secure than IPSec, however, the combination of IPSec and L2TP can be used to create a secure VPN connection.

L2TP can support these authentication methods:

- EAP (like PPTP)
- CHAP (like PPTP)
- MS-CHAP (Microsoft specific to CHAP, for remote Windows authentication. Encryption and hashing algorithms. More compatible for Windows. Doesn't require clear-text / reversibly encrypted password. Retry and password-changing mechanisms. Reason-for-failure codes if authentication fails)
- PAP (Password Authentication protocol, a very basic form of authentication. Username and unencrypted, clear-text password to be transmitted. HTTP uses this protocol. This is no longer used)
- SPAP (Shiva Password Authentication Protocol. Proprietary version of PAP. User and pass are encrypted. Susceptible to playback attacks because SPAP uses the same reversible encryption method) 
- Kerberos (Password is never sent, making it impossible for intercepting it. Username is sent, the stored password hash is used as an encryption key. The client can unencrypt that data with their password. UDP on port 88) 



