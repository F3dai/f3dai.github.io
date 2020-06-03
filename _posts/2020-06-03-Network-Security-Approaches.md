---
published: true
title: Network Security Approaches
category: Other
image: 'https://i.imgur.com/XhO3jAj.png'
author: F3dai
---

There are several ways an organisation can approach network security. A particular approach / paradigm will influence all subsequent security decisions and set the tone for the entire organisation’s network security infrastructure. 

Network security padigrams may be categorised by the following:

- **Scope of security measures taken**
- **How proactive the system is**

## Perimeter Security Approach

This approach focuses on the **perimeter** of the network. This is the philosophy of setting up functional apparatus and techniques at the perimeter of the network to secure resources and data. 

![perimeter](https://i.imgur.com/raKQy87.png)

This focus may entail **firewalls**, **proxy servers**, **limiting access** to the network, **password policies** etc. There is less effort to secure the inside of the network as the perimeter must be secured. This means that the system(s) inside the perimeter are often vulnerable.

It's vital to understand that this is a very flawed approach. However, smaller organisations may use this technique if they have a budget or inexperienced administrators.

## Layered Security Approach

This network security approach utilises a number of components to protect the system with **multiple levels** of security measures.

![onion](https://i.imgur.com/K5XQUyF.png)
[Source](https://www.plixer.com/blog/layered-security-approach/)

The technique is one in which not only the perimeter is secured, but the **individual systems within the network are also secured**. This method ensures all networking devices such as servers, workstations, routers etc within in the network are secure.

Segmenting the network and treating each subnet as **seperate** and individual with it's own perimeter security. This means that if one of the networks was compromised, not all the internal systems are affected. As a result, this is a prefered security approach. 

Administrators must take into consideration how **proactive** and/or **reactive** it is. 

A passive approach takes few or no steps to prevent an attack. A dynamic security approach, also known as a proactive defense, is one in which steps are taken to prevent attacks before they occur. For example, a system may take advantage of an IDS which aims to detect attempts to circumvent security measures. 

## Hybrid Security Approach

Networks generally fall along a continuum with elements of more than one security paradigm. It's unlikely that a network security will completely be in one paradigm. Therefore, the **combination** of the two security approaches outlined above combine to make a hybrid approach.

Hybrid Security Approaches may be prodominantly passive and layered, or it may focus more on the perimeter aprroach. 

This diagram I have created can illustrate one way to consider the network security approach:

![Cartesian](https://i.imgur.com/sziRkq3.png)

This Cartesian coordinate system considers the different factors when going for a hybrid approach. The most desirable hybrid approach is a layered paradigm that is dynamic.