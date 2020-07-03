---
published: true
title: Templed - Hackthebox Crypto
category: Writeup
author: F3dai
image: 'https://imgur.com/ltlchP7.png'
---
Templed - "I found the following message in a temple, I had the sensation that they were hiding something. Could you help me discover what it was?" This is a Hackthebox challenge under the Crypto Challenges. You must be signed up to https://hackthebox.eu to access this box.

**URL:** [Crypto Challenges](https://www.hackthebox.eu/home/challenges/Crypto)

**Difficulty:** Medium

**Author:** Xh4H

Download the archive and unzip with password "hackthebox".

<pre>mv ~/Downloads/Templed.zip . && unzip Templed.zip
hackthebox</pre>

We have a .png image.

<pre>eog Scroll.png</pre>

![Scroll.png](https://imgur.com/Y51Kves.png)

I looked up some symbol ciphers and found this:

![Google Symbols](https://imgur.com/28O279w.png)

This looks interesting, similar symbols.

### The Ciphers of the Monks

[Wikipedia](https://en.wikipedia.org/wiki/The_Ciphers_of_the_Monks)

Our "Scroll" seems to be a type of monk cipher.

*The system uses a vertical straight line as its main symbol. This symbol is essentially an axis that divides the two-dimensional plane into four quadrants.*

Here is a diagram of how each symbol is created:

![Monk Cipher](https://asecuritysite.com/public/Ciphers_clip_image002.jpg)
[source](https://asecuritysite.com/challenges/monk)

Each symbol on our cipher represents a number.

Let's take our first symbol:

![First](https://imgur.com/K0pXpgv.png)

This comprises of 70:

![70](https://imgur.com/StPxCm2.png)

And 2:

![2](https://imgur.com/B2Oqzyz.png)

So our first symbol represents the number 72. Carry this on for all of the symbols.

<pre>70 + 2 = 72

80 + 4 = 84

60 + 6 = 66

100 + 3 + 20 = 123

70 + 7 = 77

40 + 8 = 48

70 + 8 = 78

7 + 100 = 107

5 + 10 + 100 = 115

90 + 5 = 95

100 + 7 = 107

70 + 8 = 78

50 + 1 = 51

10 + 9 + 100 = 119

30 + 3 = 33

5 + 20 + 100 = 125

= 72 84 66 123 77 48 78 107 115 95 107 78 51 119 33 125</pre>

The outcome looks like a string of decimals. This has to be ASCII so let's input this on an online converter:

[ascii hex bin dec converter](https://www.rapidtables.com/convert/number/ascii-hex-bin-dec-converter.html)

![flag](https://imgur.com/pC8IsDY.png)




