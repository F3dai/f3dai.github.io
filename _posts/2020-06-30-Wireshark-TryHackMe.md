---
published: false
---
Wireshark CTFs - "Wireshark capture the flag challenges from all over the internet.. in one room" This is a TryHackMe box. To access this you must sign up to [https://tryhackme.com/](https://tryhackme.com/).

**URL:** [Wireshark CTFs](https://tryhackme.com/room/wirectf)

**Difficulty:** Medium

**Author:** ben

## Task 1 - Flag within the packets

"A CTF challenge set by csaw. During this task, you will be have to inspect a pcap file (using programs such as tshark and wireshark). You will analysis the file and release something has been... "transferred"."

Let's start this task off by downloading the file provided. We can identify this file as a "pcap capture file". 

<pre>┌─[f3dai@parrot]─[~/TryHackMe/Wireshark]
└──╼ $file net756d631588cb0a400cc16d1848a5f0fb.pcap
net756d631588cb0a400cc16d1848a5f0fb.pcap: pcap capture file, microsecond ts (little-endian) - version 2.4 (Ethernet, capture length 65535)</pre>

We can open this with Wireshark to analyse the traffic capture:

![wireshark open](https://i.imgur.com/1AglRw1.png)

If you aren't familiar with Wireshark, the main upper window displays all of the individual packets during the capture. We have an overview of the source, destination, protocol, lenght and other info.

Having a quick look, I can see some different TCP streams which means some sort of communication. between two hosts. 

The challenge question indicates something being "**transferred**". We need to look for some information being transfered.

I initially right click one of the first streams of data:

![follow TCP stream 1](https://i.imgur.com/zfxSopv.png)

Right click any packet in the stream > follow > TCP stream.

This allows for us to see an over view of the data in that stream:

![view stream 1](https://i.imgur.com/pRrMoKV.png)

Not much information is readable - in fact, the data is encrypted with TLS so we can't really understand much.

Let's try and follow another TCP stream to see if there is anything interesting. This one looked interesting as the protocol is HTTP, which means there is no encryption here. Hopefully, we will be able to identify some data:

![follow TCP stream 2](https://i.imgur.com/kJHDgXV.png)

*This is around packet 140*

Follow the TCP stream:

![view stream 2](https://i.imgur.com/OmT27o2.png)

Great, it looks like a script in Python with some mention of a flag. Let's investigate this further. To view this stream of data, you can enter the following in the filter bar at the top of Wireshark:

<pre>tcp.stream eq 4</pre>

Let's have a closer look at the code (snipped out the tailering code).

<pre>import string
import random
from base64 import b64encode, b64decode

FLAG = 'flag{xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx}'

enc_ciphers = ['rot13', 'b64e', 'caesar']
# dec_ciphers = ['rot13', 'b64d', 'caesard']

def rot13(s):
	_rot13 = string.maketrans( 
    	"ABCDEFGHIJKLMabcdefghijklmNOPQRSTUVWXYZnopqrstuvwxyz", 
    	"NOPQRSTUVWXYZnopqrstuvwxyzABCDEFGHIJKLMabcdefghijklm")
	return string.translate(s, _rot13)

def b64e(s):
	return b64encode(s)

def caesar(plaintext, shift=3):
    alphabet = string.ascii_lowercase
    shifted_alphabet = alphabet[shift:] + alphabet[:shift]
    table = string.maketrans(alphabet, shifted_alphabet)
    return plaintext.translate(table)

def encode(pt, cnt=50):
	tmp = '2{}'.format(b64encode(pt))
	for cnt in xrange(cnt):
		c = random.choice(enc_ciphers)
		i = enc_ciphers.index(c) + 1
		_tmp = globals()[c](tmp)
		tmp = '{}{}'.format(i, _tmp)

	return tmp

if __name__ == '__main__':
	print encode(FLAG, cnt=?) [ SNIPPED ]</pre>

We can see some cipher text encoded with rot13, base64 and possibly Caeser Cipher.

I'd like to use this script to decrypt the cipher. Save this to your local machine. It looks like it's written in python2.

A closer look at the script - the "cnt=?" on the last line will cause an error. I changed this to cnt=50 as that is what I saw in the encode() function above it.

How does it select which encryption algorithm to use? In the *encode()* function, the script adds a value of the corresponding cipher key to the start of the string.

In fact, these are the keys:

- 1 : rot13
- 2 : base64 encode
- 3 : caesar cipher

Let's start by running this script with python / python2 and see what it gives us:

<pre>python2 script.py</pre>

![run script](https://i.imgur.com/KCBmvdF.png)

It's given us some cipher text. Let's try and decrypt this given the order of encryptions from the script.

**Caeser (shift=3) > base64 > Rot13**

It looks like the script has a list of the decryption algorithms:

<pre># dec_ciphers = ['rot13', 'b64d', 'caesard']</pre>

Let's uncomment this and change our functions to decrypt instead of encrypt:

**rot13()**:

This function performs ROT13 encryption. According to the 2 alphabets, this translates each character 13 places forward. For example, A -> N, B -> O etc. Let's change this so it simply goes back 13 places. I just swapped the two strings around:

<pre>def rot13(s):
	_rot13 = string.maketrans( 
	"NOPQRSTUVWXYZnopqrstuvwxyzABCDEFGHIJKLMabcdefghijklm",
    	"ABCDEFGHIJKLMabcdefghijklmNOPQRSTUVWXYZnopqrstuvwxyz")
	return string.translate(s, _rot13)</pre>
    
**base64()**:

I simply added this function:

<pre>def b64d(s):
	return b64decode(s)</pre>
    
**caeser()**:

Similar to ROT13 - this function translates each character 3 places forward, so I changed the *shift=* parameter from 3 to -3. 

<pre>def caesar(plaintext, shift=-3):</pre>

Now let's add an encrypt() function:

**decode()**:

This part is important, we need this to decode our given string. Here is my code:

<pre>def decode(cipher):
    print "Decoding... \n"
    while "flag" not in cipher:
        i = int(cipher[0])
        raw = cipher[1:]
        print "Running function " + dec_ciphers[i-1]
        _cipher = globals()[dec_ciphers[i-1]](raw)
        cipher = _cipher
    print cipher</pre>
    
This will essentially run through our decryption functions and test to see which is the correct one to use. 

The while loop will constantly run until the plaintext has been found in the ciphertext / plaintext.

The *cipher[1:]* sanitises the cipher, it removes the first number added previously as mentioned earlier.

The cipher text is then run through the corresponding decryption algorithm, and repeated through the while loop process. We can display all the different functions the cipher text passes through by adding the *print "Running function " + dec_ciphers[i-1]* statement.

Just to test if this works, I've adjusted the main section too:

<pre>if __name__ == '__main__':
	print "Encoding... \n"
    cipher = encode(FLAG, cnt=50)
    
    print decode(cipher)</pre>
    
So this script should encode the *FLAG* variable then decode it. Let's run this:

<pre>python2 script.py</pre>

Result:

![script result](https://i.imgur.com/JqTnEAF.png)

Great, our script works. So instead of encrypting and decrypting the FLAG variable they provided, let's plug in our cipher that we found in the packet capture inside the TCP stream. 

I made a global variable called cipher with our cipher text and commented out the encoding parts of the main function:

<pre>cipher = '2Mk16Sk5iakYxVFZoS1RsWnZX [ SNIPPED ] ' 

...

if __name__ == '__main__':
    #print "Encoding... \n"
    #cipher = encode(FLAG, cnt=50)
    print decode(cipher)</pre>
    
Let's re-run the script:

![Flag 1](https://imgur.com/tN3TiJo.png)

Our script has successfully decoded the flag.

Here is the script I used:

[Pastebin - Wireshark Python Script](https://pastebin.com/bjWfC444)

Moral is to never visit HTTP websites! Always HTTPS.

## Task 2 - Halloween PCAP Challenge from Cloudshark 

"In this challenge you will be analysing a pcap file using Wireshark, looking for 5 hidden 'pumpkins'.

This challenge was credit by cloudshark"

We have another traffic capture file which we need to investigate.




