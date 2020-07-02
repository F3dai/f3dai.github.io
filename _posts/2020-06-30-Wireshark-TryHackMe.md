---
published: true
category: Writeup
title: Wireshark CTF - TryHackMe Walkthrough
author: F3dai
image: 'https://imgur.com/sUbOkno.png'
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

Now let's add a decrypt() function:

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


### When you find the first (most obvious) ascii pumpkin, what was the destination IP address? 

This first question is asking us for information about the "obvious" ascii pumpkin. Since this is apparently obvious, I try scanning throught the packets, paying attention to the ascii text in the data. 

Since most of the connections are through HTTP, we can see all the unencrypted websites and data transfered.

This packet caught my eye:

![ascii](https://imgur.com/qaayCHF.png)

The user visited a website:

<pre>[Full request URI: http://Thank you for visiting https://asciiart.website//]</pre>

### Download all images found in the pcap file. What is the name of the pumpkin image?

As mentioned before, there is a lot of HTTP traffic which *dangerously* allows us to see all the data transfered, including images, videos etc.

Wireshark has a function to scan and identify all images found during the packet capture. 

Go to File > Export Object > HTTP

![Export Objects Setting](https://imgur.com/P00xtEL.png)

Wireshark extracts all the files:

![Extract files](https://imgur.com/Uzj9sVP.png)

Let's save all of these to our local machine, I removed all the non-image files by creating a tmp directory, moving all the images there and deleting the rest of the files:

<pre>mkdir tmp && mv *.GIF *.gif *.png *.jpg  tmp/
rm * -f && mv tmp/* .&& rm tmp/</pre>

Now we have all our images in the current direcotry, have a look at each image:

<pre>eog *</pre>

I couldn't view this image:

![Wrong format](https://imgur.com/pAiNkWr.png)

<pre>Not a JPEG file: starts with 0x89 0x50</pre>

I checked out the magic number / file signature and turns out it's a .png file so let's change the format and try again.

<pre>head jack-o-lantern.jpg
mv jack-o-lantern.jpg jack-o-lantern.png
eog jack-o-lantern.png</pre>

![File signature](https://imgur.com/PHbZO1y.png)

Turns out this is our pumpkin:

![Pumpkin lantern](https://imgur.com/jMvMQZu.png)

### Find the pumpkin that on TCP port 666. Whats the main character that makes the pumpkin up? 

Thanks to Wireshark's filter functionality, we can easily display packets on TCP port 666:

<pre>tcp.port == 666</pre>

I had a scroll through the packets and found this strange looking packet:

![strange???](https://imgur.com/mFPVWnm.png)

Follow the TCP stream and you will find the third pumpkin.

![Pumpkin 3](https://imgur.com/aOvBX0G.png)

### Find the pre-master token and decrypt the traffic. What the file data size of this next pumpkin (in bytes)?

Before starting this, I *highly* reccommend you read this article about SSL / TLS handshake to understand the different steps when authenticating a user, using a "pre-master token".

[TLS / SSL Handshake - CyberGoat](/other/TLS-Handshake/)

*TLS is an encryption protocol designed to secure Internet commonunications. As for any TCP communication, there is a handshake. The TLS handshake is the process which initiates the communication sessions with TLS encryption.*

During the handshake, the user and server exchanges the "premaster secret":

*The premaster secret: Client sends another string of random bytes – “the premaster secret”. This can only be decrypted with the private key by the server.*

I know that HTTPS operates on port 443, I have identified the encrypted HTTPS traffic by using the filter:

<pre>tcp.port == 443</pre>

If you have a look at some of the application data, it is evidently unreadable:

![Unreadable SSL traffic](https://imgur.com/A06YBO8.png)

We need to find the pre-master token to be able to read this. Let's logically assume that it has been exchnaged before this instance. 

To identify all the conversations on a packet capture, go to Statistics > Conversations > TCP.

![Conversations](https://imgur.com/GxkONsV.png)

On the TCP tab, you can see all the conversations between clients:

![TCP tab](https://imgur.com/wfHq2D5.png)

Most of them are on port 80. It's unlikely the key would have been shared over a web page of some sort but the conversation on port 25 looks interesting. SMTP operates on this port, maybe they exchanged the key over e-mail?

Right click this and find this on the packet capture, or just look for SMTP traffic:

![Right Click Find](https://imgur.com/fs7bBgV.png)

Follow the SMTP TCP stream when you find it, you will see something like this:

![SMTP e-mail](https://imgur.com/EoIVfJW.png)

<pre>Subject: MICHAEL HAS ESCAPED!

Laurie! Michael has escaped the sanitarium and is in the house! Use this to find him:

CLIENT_RANDOM 4CD4ADF90628A9AFB29D50F093A5FAD4FC09CCF3F173E52F7B2390573989659F E8AC4AFFCDAD005F5ED4E29D2625A49378A25E7D5B85D5418AC51C1D0CC50B52B39DB3998C606202339178C1EA441CE0</pre>

Fortunately, Wireshark has a function to include a key for decrypting traffic so let's save this key to a file:

<pre>echo "CLIENT_RANDOM 4CD4ADF90628A9AFB29D50F093A5FAD4FC09CCF3F173E52F7B2390573989659F E8AC4AFFCDAD005F5ED4E29D2625A49378A25E7D5B85D5418AC51C1D0CC50B52B39DB3998C606202339178C1EA441CE0" > key.txt</pre>

Now add this by going to Edit > Preferences > Protocols Dropdown > TLS then import your key:

![Import Key](https://imgur.com/SScBwmU.png)

Go back to the SSL traffic on port 443, you will see a 2 new HTTP packets:

![HTTP SSL](https://imgur.com/iqKSYGU.png)

The user requested the file /michael.txt. Analyse the file information by looking under "Hypertext Transfer Protocol".

### Extract the RTP stream. What is the audio file from?

What is RTP?

*The Real-time Transport Protocol (RTP) is a network protocol for delivering audio and video over IP networks. RTP is used in communication and entertainment systems that involve streaming media, such as telephony, video teleconference* - [Wikipedia](https://en.wikipedia.org/wiki/Real-time_Transport_Protocol)

If the traffic isn't encrypted, we can extract this to rebuild an audio file. I looked into the Wireshark function where you can automatically identify the RTP data but no luck.

In short, when communicating over a network, it is either send over TCP or UDP protocol on the Transport Layer (OSI Ref). Both have benefits and disadvantages but I won't go into it too much as this isn't a networking lesson. TCP is great for reliable communication because it confirms everything is correctly sent (Handshakes, ACK packets etc), for example, e-mailing, viewing web pages etc whereas UDP is great for fast communication that need a continuous flow like streaming media, calling etc because it does not check if every packet has been received (so there will be some packet loss).

For this reason, let's look for UDP packets - I sorted the traffic by protocol to find UDP at the top. 

As you can see, Wireshark has identified these as just UDP datagrams, not RTP. Therefore, we need to decode this as RTP. Right click a packet in the stream and select decode as:

![Decode as](https://imgur.com/9vBfR2k.png)

![Decode Box](https://imgur.com/Hj7hHZV.png)

You will be prompted with a box which has *UDP* for Field and *(none)* for Current. We need to change the current to RTP. Use the dropdown to find RTP and save this.

You may have noticed all the UDP traffic on Wireshark changed to RTP protocol:

![UDP to RTP](https://imgur.com/poFIoiZ.png)

We can now go to Telephony > RTP > RTP Streams to view the RTP data.

![Select RTP](https://imgur.com/FFK66rr.png)

You should see something like this:

![View RTP](https://imgur.com/85gkqmG.png)

Right click the RTP Stream it found and go on "Analyse". This will bring you to a new window that shows a lot more data about the RTP stream Wireshark identified. 

![RTP data](https://imgur.com/dTjAWfD.png)

We have the option to Play Stream (bottom right).

![Audio](https://imgur.com/8DBbvkD.png)

This is from Great Pumpkin, Charlie Brown.
