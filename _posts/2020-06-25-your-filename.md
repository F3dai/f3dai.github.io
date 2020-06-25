---
published: false
---
Crack the hash - "Cracking hashes challenges. Can you complete the level 1 tasks by cracking the hashes?" This is a TryHackMe box. To access this you must sign up to [https://tryhackme.com/](https://tryhackme.com/).

**URL:** [Crack the hash](https://tryhackme.com/room/crackthehash)

**Difficulty:** Easy

**Author:** ben

This challenge focuses on cracking hash values. We will start off with task 1:

## Task 1

We are given a few different hash values, I have compiled and listed them here:

<pre>48bb6e862e54f2a795ffc4e541caed4d
CBFDAC6008F9CAB4083784CBD1874F76618D2A97 
1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032
$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom
279412f945939ba78ce0758d3fd83daa</pre>

So the first step is to identify each hash. For convenience, I added these to a file called "hashlist"

<pre>nano hashlist
[enter all hash values line by line]</pre>

I personally find hash-identifier really useful in these sort of challenges - this comes pre installed on Kali Linux but my machine doesn't have it. You can install the script here:

[hash-identifier - blackploit](https://github.com/blackploit/hash-identifier)

You may install the script using this command:

<pre>wget https://raw.githubusercontent.com/blackploit/hash-identifier/master/hash-id.py</pre>

You could use this for each hash in the challenge (manually) but I wanted to a way to scan through the file I created and to spit out the results.

This is a bit extra, you don't need to do it but I always try to take the challenge more interesting.

I edited the script to just return the possible hash types - that's all. This meant removing the while loop to avoid persistant input, removing the logo, removing the least possible hash results.

Here is a pastebin of the script:

[Pastebin - hash-id modified](https://pastebin.com/7918xsRW)

You can choose to install and rename my modified version of the script with the following command:

<pre>wget https://pastebin.com/raw/7918xsRW
mv 7918xsRW hashid.py</pre>

Now let's make a loop in shell to test out every line of the file. We can use "cat" to print out every line and pipe to variable "$i". 

<pre>cat ~/TryHackMe/CrackTheHash/hashlist | while read i
do echo $i
python3 hashid.py $i
done</pre>

The above commands essentially assign each line to the variable "i", which we can use in our script inside a while loop.

Here is the following result:

![hash results part 1](https://i.imgur.com/61iEZOm.png)

<pre>cat ~/TryHackMe/CrackTheHash/hashlist | while read i
> do echo $i
> python3 hashid.py $i
> done
48bb6e862e54f2a795ffc4e541caed4d
Possible Hashs:
[+] MD5
[+] Domain Cached Credentials - MD4(MD4(($pass)).(strtolower($username)))
CBFDAC6008F9CAB4083784CBD1874F76618D2A97
Possible Hashs:
[+] SHA-1
[+] MySQL5 - SHA-1(SHA-1($pass))
1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032
Possible Hashs:
[+] SHA-256
[+] Haval-256
$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom

 Not Found.
279412f945939ba78ce0758d3fd83daa
Possible Hashs:
[+] MD5
[+] Domain Cached Credentials - MD4(MD4(($pass)).(strtolower($username)))</pre>

We've identified all but one of the hash values. This means we can go ahead and decrypt the known ones with online tools:

**48bb6e862e54f2a795ffc4e541caed4d - md5**:

[md5 decrypt](https://md5decrypt.net)

![md5](https://imgur.com/3NI4FdP.png)

**CBFDAC6008F9CAB4083784CBD1874F76618D2A97 - SHA-1**:

[SHA-1 decrypt](https://sha1.gromweb.com)

![sha1](https://imgur.com/undefined.png)

**1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032 - SHA-256**:

[SHA-256 decrypt](https://hashtoolkit.com/reverse-sha256-hash/)

**$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom - Not Found**:

The script was unable to identify this one. Some research shows this is a bcrypt ($2*$), Blowfish (Unix) hash type. 

Using the [hashcat modes](https://hashcat.net/wiki/doku.php?id=hashcat), we identify mode 3200 for Blowfish. I inserted this hash type in a seperate file and executed a hashcat command to crack it:

<pre>echo $2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom > blow
</pre>



