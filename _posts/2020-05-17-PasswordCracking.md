---
published: true
Title: Password Cracking
category: Ethical
title: Password Cracking
image: https://imgur.com/xRXW0Xj.png
---

This is an exciting and important topic to understand for everyone. It is vital to understand how adversaries may utilise the various ways of cracking passwords outlined in this article to gain unauthorised access to an account or system.

A password is essentially a basic security mechanism that consists of alphabetic, numeric, alphanumeric, and symbolic characters, or a combination of them all. Passwords are a form of authentication to determine you are the correct owner / authorised person to access whatever is being accessed. It is a method of keeping something restricted.

Password cracking may refer to numerous methods utilised to discover a computer password. Password cracking is an extremely popular and unethical method of gaining access to computers as it usually serves as the only method of authentication. Once a password for a high-level user is cracked, there is no longer a need for finding other vulnerabilities and exploits. These further exploits will not be discussed in this article for this reason, but they are covered in depth in the writeups on this website.

The CTF-style writeups found on this website cover many topics of finding vulnerabilities and exploitation including password cracking. For example, /DerpNStink or /HackFest.
There will be four major categories of password cracking methods illustrated below with some examples too.

- Dictionary Attacks
- Brute Force Attacks
- Hybrid Attacks
- Rainbow Tables

Before going into detail about each attack it is important to understand how passwords work and how they are stored.

## Password Hash

Organisations will hopefully store your passwords as a hash, this is just a way to represent the data as a unique string of characters and symbols. Hashing is simply a one-way mathematical algorithm producing a unique output that represents your password. I’ll show you an example. I’ll hash the following passwords with the MD5 algorithm:

-	"Password"
-	"password"

Through my Linux machine I can easily output this in my terminal:

![echo](https://imgur.com/YYsDj6I.png)

<pre>echo -n "Password" | md5sum
echo -n "password" | md5sum</pre>

<pre>dc647eb65e6711e155375218212b3964
5f4dcc3b5aa765d61d8327deb882cf99</pre>

As you can see, this has produced two completely different unreadable hash values despite the small change in the lowercase and uppercase p. There is now no algorithm that can change that back to the original string “password” or “Password”. However, you may identify this hash as “Password” if you know the word and run it through the same hash function. This is cracking.

When trying to understand how you can crack passwords, it is important to understand what a login process is like. When you sign into a website your password taken and processes through an algorithm like the one outlined above and stored in a database. For example, this is a php script that takes the password given and turns it into a hash before storing it on a database.

<pre>$Password = MD5($_POST[‘password’]);</pre>

So how does the website know the password you are entering is the correct on if it stores your hashed password?

As mentioned before, a possible way to verify the hash belongs to your password is to convert the given password to the hash and compare them. Take this script below (pseudo-code):

<pre>If (md5($Submitted_Password) == $Stored_Password_Hash) ThenLogin()</pre>

The given password is hashed, then compared to the stored password. If they are the same then the user has been authenticated.

## Password Salt

The hash function described above has become quite problematic nowadays as it is shown to be insecure. Md5 allows for fast brute force attacks (we will look into this more), md5 is also so widely used even now so using dictionaries provides an easy method around this, and also the cryptographic method is problematic as it has a very low collision resistance, meaning the same passwords will have the same hash values. A solution to this is using password salts.

A password salt is another string that is added to a user’s password before it is encrypted. This string could be anything like the user’s name (bad practice) or something completely random. This makes the passwords significantly harder to crack as it makes every password unique. Let us do a really simple example below. We have previously identified that “password” has the md5 hash value of:

<pre>5f4dcc3b5aa765d61d8327deb882cf99</pre>

Below I will show how using salts can make the same passwords, unidentifiable by their hash values. The following command will produce a completely random string of letters and numbers:

<pre>head /dev/urandom | tr -dc A-Za-z0-9 | head -c 13 ; echo ''</pre>

<pre>ofAtzK9oX6yMk
I5EH4Oxkf7gAx</pre>

![salts](https://imgur.com/VejwSfi.png)

Now, these two salts will be combined with “password” and then converted into an md5 sum. These are the hash values:

<pre>3f36235ab6cecde6f4ada57c15c3b006
805147566a8495dfb1119dcf1a0b658d</pre>

![hashsalts](https://imgur.com/T1LE5IS.png) 

This also dramatically reduces the chances of password collisions creating a unique hash value than before, even if the passwords are the same. This will make it much harder for an attacker to guess your password using traditional brute-forcing methods.

## Online and Offline Password Cracking

Now we can start looking at how passwords are cracked! When performing a password cracking attack, it can be either online or offline. Online attacks are when attackers do not have access to the password hashes. This typically involves a web login page and trying to guess the password, hoping for a positive response. Online attacks are very noisy and slow. Now adays website use a lockout feature to prevent many password guesses, making much harder for attackers. You could try and use proxies to mask up your IP to get more attempts, but this takes up more resources.

Offline attacks are made possible when an attacker has a password hash. Instead of attacking an online web form you could carry out this attack on your local machine. Thanks to this, you have all the resources and power that your computer has making it much quicker with no limits such as lockouts.

## Password Cracking

For reference, I will be using my Kali Linux OS running as a virtual machine. Kali Linux is usually my go-to operating system as it is tailored for penetration testing with many pre-installed tools.

## Dictionary Attack

Dictionary attacks utilise a dictionary where every word is used to guess the password against the hash. 

These dictionaries may include different languages and common passwords to save time. These types of attacks are used when trying to exploit weak passwords (commonly used). As a result of this laziness, dictionary attacks are often quite successful. 

Here is an example of a dictionary attack on an intentionally vulnerable machine “Metasploitable2”. In this scenario, we are presented with a web login on DVWA (Damn Vulnerable Web Application). 

1. Make sure you have DVWA set up to low. 

![dvwa](https://imgur.com/l8vJ4nb.png)

2. Go to the Brute Force page. 

![dvwa Brute](https://imgur.com/68dQnNZ.png)

3. Set up a web proxy, this is on Firefox. And open Burp Suite. Use these settings exactly!

![proxy](https://imgur.com/AGC5oGz.png)

4. Switch intercept on in Burp Suite (should already be on). 
Proxy > Intercept 

![intercept](https://imgur.com/Cn4mt94.png)

5. Make a request – enter in some credentials and analyse the intercepted traffic. Make note of this request, we will use it in a later step.

![intercepted](https://imgur.com/IU829SC.png)  

6. Forward the traffic, and stop the intercept. Make not of the failed login message. “Username and/or password incorrect.”

![failed msg](https://imgur.com/wEPhtZB.png)

7. Now construct the hydra command:

<pre> hydra [host] -l [user] -P [passwordlist] http-get-form “ [username ^USER^ password ^PASS^ ] : F=[<Failed Message] : H=Cookie: [Cookie Value]” </pre>

<pre> hydra 192.168.56.118 -l admin -P "/usr/share/wordlists/rockyou.txt" http-get-form "/dvwa/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:F=Username and/or password incorrect.:H=Cookie: PHPSESSID=b39e1a2e30e1ce45c5927951e84aa52c" </pre>

This command is first specifying our host IP, -l user, -P wordlist, http-get-form is the type of traffic (this is a GET request). "/login/page : username and password field : F = Our message we noted if login is incorrect : H=Cookie: Our cookie from this session". Run this command on your machine, you can include -V as well to see the verbose output to see what's going on.

I have used a premade wordlist in Kali Linux, but you can use anything you like. This is the output: 

![dictionary](https://imgur.com/G1gxpfG.png) 

The password “password” has been found by looking at the responses of the web requests made by hydra. Hydra keeps testing passwords until it receives anything but the failed login message we specified.

## Brute Force Attack

A brute force attack aggressively attempts every possible combination in a range of characters. When creating a range of characters, attackers may use for example alphanumeric, lower case, uppercase and key spaces. This method will typically take much longer than a dictionary attack but covers all possible combinations that a dictionary attack may not have.

In this example, I will create use the md5 hash value for password: 

<pre>5f4dcc3b5aa765d61d8327deb882cf99</pre>

I will save this to a file named “hash” with the following command:

<pre>echo "5f4dcc3b5aa765d61d8327deb882cf99" > hash </pre>

Let us use a tool called Crunch to create our brute force attack wordlist. <min> and <max> is the length of characters that are being specified, and the -o is the file to output the words to.

<pre> Crunch [min] [max] [charset] -o [output] </pre>

In this example, I am taking a “guess” that it is between 8 and 8 characters. Just to show the mass possibilities, using the full alphabet for just 8 characters results in a file around 1TB in size.

<pre>crunch 8 8 abcdefghijklmnopqrstuvwxyz -o wordlist.txt</pre>

Produces:

![1tb](https://imgur.com/ebNAI94.png)

Let us pretend we got some intel that the password only includes the characters “adoPrsw” for the sake of time.

<pre>crunch 8 8 adoPrsw -o wordlist.txt</pre>

![8-8](https://imgur.com/vvG7Wp5.png)

Now we have a file called wordlist.txt, this is a snippet of the file: 

![wordlist](https://imgur.com/KubdfwB.png)

We can try and brute force the password hash. We know that the password is in there somewhere.
Let’s use John to try and crack this hash in our file with the following command.

<pre>john --format=raw-md5 hash --wordlist wordlist.txt</pre>

We need to tell John that the hash is md5 with --format. We then specify the file with the hash value inside, ours is “hash” and finally our wordlist is specified with --wordlist, wordlist.txt is the wordlist we created with crunch.

Almost immediately john has cracked this hash.

![john](https://imgur.com/N3PsN0z.png)

## Hybrid Attack

A hybrid attack is a mixture of both a dictionary and brute force attack. Attackers can provide a password wordlist and an additional brute-force attack to each word in the wordlist.  A good situation where this kind of attack would work great is where you are looking for variations of a word. For example, my old school would provide us with a password of our name and some numbers. Knowing this, I could use a wordlist of all names in the school and brute the numbers easily, quickly allowing me to crack every single student’s password in the school.

For this example, let us create a few different passwords based on common names with a completely random combination of numbers after and get the md5 hash values for them.

<pre>2c6768550babedb9e5e61f53fc555b78  - John37985
bc09cd617a0744b58871f0b38bb070d4  - Max65613
1a6ef1bb49b169dc44bc4c1be2e22fb5  - Admin74595
12dc4c70158b6720ab1dc79fdd8c2298  - Charlie92309
f5bd3fe44f6c5547bbe3b408c0cea0a9  - Robert34834</pre>

I have stored these in a file called “hashz”.

![hashz](https://imgur.com/KjOvLTW.png)

Let’s get a wordlist of common names / usernames: 

![names wordlist](https://imgur.com/ZiblCu5.png)

And I will create another wordlist with numbers 5 characters long with just numbers using crunch again:

<pre>crunch 5 5 1234567890 > numbers.txt</pre>

We will be using Hashcat, an advanced password “recovery” tool. This is the following command I will use:

<pre>hashcat -m 0 -a 1 hashz names.txt numbers.txt</pre>

-m 0 is the type of hash we are using (md5), -a 1 is the attack type which is combination. “hashz” is the file with the hash values, names.txt is the list of common names and numbers.txt is a list of random numbers created with crunch.

![hashcat](https://imgur.com/vIRlOZA.png)

This means that given we knew about the format of passwords (like my old school), we could crack these 5 hash values in only 15 milliseconds!

For this reason, hybrid attacks are extremely useful when you have an idea of how passwords are formatted.

## Rainbow Tables 

Rainbow tables are lookup tables which contain a huge amount of possible password combinations in a given character set. It is essentially a wordlist of a completed brute force attack made to be reused without having to re-convert all the passwords to hash values every time. This saves a significant amount of time attackers can save up to 99% of their time by using rainbow tables. 

[Wikipedia](https://en.wikipedia.org/wiki/Rainbow_table)

Rainbow tables can, however, get extremely big. To illustrate, here is an example of the size of tables from RainbowCrack Project:

![exampletable](https://imgur.com/69IGXjB.png)

To get involved with password cracking with Rainbow Tables, there are some easy to use tools you can use on Linux.

You'll need to download RainbowCrack and install tables from the following website:

[Rainbow Crack](https://project-rainbowcrack.com/table.htm)

Make sure to have at least half a Terabyte free. As I mentioned before rainbow tables can be extremely large but make cracking a speedy process.
