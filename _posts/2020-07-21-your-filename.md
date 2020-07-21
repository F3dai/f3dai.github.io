---
published: false
---
Let's enumerate our first port - 80:

![port 80](https://imgur.com/xT4s0Tg.png)

We are asked for some credentials but I don't have any so far.

Let's move onto the next port - 139 and 445. Since this machine is using smb, my spidey senses told me to use enum4linux. I wanted to get a list of shares so I used the parameter -S:

<pre>enum4linux -S</pre>

![enum4linux](https://imgur.com/qXWr8Rr)

And to enumerate users:

<pre>enum4linux -U</pre>

