---
published: true
title: TLS / SSL Handshake
image: 'https://imgur.com/4LHhRL4.png'
category: Other
author: F3dai
---

TLS is an encryption protocol designed to secure Internet commonunications. As for any TCP communication, there is a handshake. The TLS handshake is the process which initiates the communication sessions with TLS encryption. 

TLS handshakes provide the fundamental part of how HTTPS works.

Secure Sockets Layer (SSL) was initially developed for HTTP as the encryption protocol. TLS replaced SSL a while ago – when people talk about SSL handshakes, they are often refering to TLS handhsakes.

Whenever a user communicates with a web page over HTTPS, a TLS handshake will take place. In fact, these handshakes will take place with any communication over HTTPS, for example, API calls, DNS with HTTPS etc.

## What happens during a TLS Handshake?

![General Handshake TLS SSL Cloudflare](https://www.cloudflare.com/resources/images/slt3lc6tev37/5aYOr5erfyNBq20X5djTco/3c859532c91f25d961b2884bf521c1eb/tls-ssl-handshake.png)
[Source](https://www.cloudflare.com/learning/ssl/what-happens-in-a-tls-handshake/)

During the communication between a client and server, the following information will be negotiated:

-	Specify TLS version
-	Decide Cipher suites
-	Authenticate using public key and SSL certificate authority’s digital signature
-	Create session keys for symmetric encryption after handshake

It’s important to note that each step may vary depending on the kind of key exchnage algorithm as well as the cipher suites. 

1.	‘Client Hello’: Client says hello to initiate the handshake. This includes TLS version, cipher suite, and a string of random bytes – the “client random”
2.	‘Server Hello’: The server responds with it’s SSL certificate, cipher suite, “server random” and an additional string of random bytes.  
3.	Authentication: Client verifies SSL certificate with the certificate authority, confirming the server is who they say they are. 
4.	The premaster secret: Client sends another string of random bytes – “the premaster secret”. This can only be decrypted with the private key by the server. 
5.	Private key used: Server decryptes premaster secret.
6.	Session keys created: Client and server generate sessions keys using the client and server randoms, and the premaster secret. 
7.	Client ready: “Finished” message sent with session key encrypted in it.
8.	Server ready: “Finished” message sent with session key encrypted in it.
9.	Encryption: Secure symmetric encryption achieved – communication contunues with session keys. 

A cipher suite is a set of encryption algorithms. There are many different algorithms and cipher suites, this makes up the essential part of a TLS handshake where both the client and server negotiates. 
