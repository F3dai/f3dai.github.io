---
published: true
title: The Impossible Challenge - TryHackMe Walkthrough
category: Writeup
image: 'https://imgur.com/dkE5Kgl.png'
author: F3dai
---
Impossible Challenge - "Download the file, and find the Flag! - Hmm" This is a TryHackMe box. To access this you must sign up to [https://tryhackme.com/](https://tryhackme.com/).

**URL:** [The Impossible Challenge](https://www.tryhackme.com/room/theimpossiblechallenge)

**Difficulty:** Medium

**Author:** 0day

We are initially given this cipher:

<pre>qo qt q` r6 ro su pn s_ rn r6 p6 s_ q2 ps qq rs rp ps rt r4 pu pt qn r4 rq pt q` so pu ps r4 sq pu ps q2 su rn on oq o_ pu ps ou r5 pu pt r4 sr rp qt pu rs q2 qt r4 r4 ro su pq o5</pre>

In this challenge, I will be using CyberChef:

[CyberChef](https://gchq.github.io/CyberChef/)

This is a useful website which can help you identify and decrypt ciphers.

The cipher has been encrypted a few times - I attempt to decrypt with ROT13 and ROT47 and get this result:

![rot 13 rot 47](https://imgur.com/bYBZ1i4.png)

This now looks like hex as they are double digit values (0 - 9 and a - f). 

You can convert from hex but I was lazy and used the "magic" option which just completed the rest of the decryption. This is what I got:

![plain text](https://imgur.com/Od4X139.png)

After decoding from hex, then from Base64:

<pre>It's inside the text, in front of your eyes!</pre>

This looks like a hint - something is inside the text, in front of our eyes, implying the next "thing" is somewhere we have already been.

We've only really visited CyberChef and the TryHackMe page as well as the locked zip file. The only place I believe we can find some more information is the TryHackMe Page.

There seems to be nothing in the Task 1 section of the web page:

![task 1 page source](https://imgur.com/aFRKeVM.png)

However, there is something strange about the challenge description "Hmm" below the title:

![Hmm](https://imgur.com/rYDNJJp.png)

Attempting to highlight the red dots shows a pattern, let's further investigate this.

This looks like 0 Width Spaces.

[Zero width space - Wiki](https://en.wikipedia.org/wiki/Zero-width_space)


A quick search of a zero width space cipher reveals a type of steganography called **Unicode Steganography with Zero-Width Characters**

I used this website to decode the cipher:

[Unicode Steganography](https://330k.github.io/misc_tools/unicode_steganography.html)

This is my result:

![decrypt](https://imgur.com/4uyEw1I.png)

<pre>password is h******z</pre>

I use this to unzip the archive provided by the challenge:

<pre>unzip Impossible.zip
h******z</pre>

We are given our flag:

<pre>THM{Z************************Z}</pre>

![done](https://imgur.com/YlmhVDj.png)








