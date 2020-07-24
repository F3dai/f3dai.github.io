---
published: true
category: Other
title: The Caeser Cipher
author: F3dai
image: https://imgur.com/LtdjX4l.png
---
# The Caeser Cipher

This article will be outlining what the Caeser Cipher is and how to encrypt/decrypt using the Caeser Cipher tool from Github.

## What is the Caeser Cipher?

The Caesar cypher is known as one of the earlier and simplest cyphers. It is a type of **substitution cypher** in which each letter in the **plaintext** is **'shifted'** a number of places across the alphabet. 

Let's take a look at the following diagram:

![cipher diagram](https://upload.wikimedia.org/wikipedia/commons/thumb/4/4a/Caesar_cipher_left_shift_of_3.svg/1200px-Caesar_cipher_left_shift_of_3.svg.png)
[source](https://en.wikipedia.org/wiki/Caesar_cipher)

This demonstrates how the "plaintext" letters can be manipulated by shifting each value so it is represented by a completely different letter.

When encrypting a message using this algorithm, a key is used to indicate the "shift". In this case, we will use a key of 3.

![cipher example](https://camo.githubusercontent.com/b74e7a81cbde201cad7f91ca0c5423d83bf24641/68747470733a2f2f6e696b68696c6d6163686368612e66696c65732e776f726470726573732e636f6d2f323031352f30392f6361657361722d6369706865722e6a7067)
[source](https://github.com/rickdaalhuizen90/caesar-cipher)

Here, the message **TREATY IMPOSSIBLE** has been encrypted to something completely different.

## Caeser Cipher Tool

I have created a lightweight tool that performs encryption and decryption using Caeser Cipher. Follow this link to view the repository:

### [Caeser Cipher - Github](https://github.com/F3dai/caeser-cipher)

Please note, I will be demonstrating this script on Linux - Parrot OS. To install the script to your local machine, execute the following command:

<pre>wget https://raw.githubusercontent.com/F3dai/caeser-cipher/master/julius.py</pre>

To run the script, simple execute this command:

<pre>python3 julius.py</pre>

<pre>┌─[user@parrot]─[~/caeser]
└──╼ $python3 julius.py 

[SNIPPED]

# Select menu item #

[1] Encrypt
[2] Decrypt
[3] Quit

Please enter your choice:</pre>

![Menu](https://imgur.com/LtdjX4l.png)

### Encrypt

To encrypt a message, simply select the first menu item and enter the parameters.

You will be asked for the shift key, this will be the number of places you want your letters to be shifted across the alphabet. _It must be in the range -26 to +26_.

Then enter your desired message you want to encrypt.

<pre>Please enter your choice: 1
[*] Shift key (-26 - 26): 15
[*] Message: Hello friend

[+] wtaad ugxtcs</pre>

We have our cypher text: **wtaad ugxtcs**

![encrypt](https://imgur.com/9qS0Gyp.png)

### Decrypt

To decrypt a message, select the second menu item and enter in your ciphertext.

We will be using the ciphertext: **wtaad ugxtcs**

<pre>Please enter your choice: 2

[*] The longer the cypher, the more accurate the results
[*] Enter cypher: wtaad ugxtcs

## Most likely messages ##

[+] ebiil cofbka
[+] hello friend
[+] khoor iulhqg
[+] qnuux oarnwm</pre>

![decrypt](https://imgur.com/9qS0Gyp.png)

The script has returned a selection of the more likely plaintext by **brute-forcing** and **analysing** the words. 

As we can see, our original plaintext has been identified:

<pre>[+] hello friend</pre>

