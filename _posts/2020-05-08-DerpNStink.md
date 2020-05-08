---
published: false
---
**URL:** [DerpNStink](https://www.vulnhub.com/entry/derpnstink-1,221/)

**Difficulty:** Beginner

**Author:** Bryan Smith

### Enumeration

Set up the Machine with a host-only adapter and run a net discover command to find the associated IP. My interface eth1 is Host Only on my main OS (Kali Linux).

<pre>netdiscover -i eth1</pre>

![net](https://imgur.com/FMu22dc.png)

The IP 192.168.56.103 has been found so we will do a port scan with the following IP:

<pre>nmap -sS -A -p- -o "nmap" 192.168.56.103</pre>

![nmap](nmap -sS -A -p- -o nmap 192.168.56.103)

The first port that is open is the FTP. Sometimes users can log in as Anonymous with no password needed. 

<pre>ftp 192.168.56.103
Anonymous</pre>

![ftp](https://imgur.com/6lgyL35.png)

This was not possible so I will move onto enumerating the next port.

Port 80 is open so have a look on a browser. This is what the web page looks like:

![web](https://i.imgur.com/35zMoqV.png)

Nothing obvious sticks out so look at the source code.

![source](https://imgur.com/eWpQyFN.png)

There is an interesting link worth checking in one of the < script > tags as shown below. There is a link to /webnotes/info.txt. Go to that directory.

![info](https://imgur.com/h4EluwO.png)

There is a message to "stinky" (as possible user) to update the local hosts file with a local dns so the blog website can be accessed. There was no blog seen initially so let's go ahead and update our host file on our local machine. Before this, I want to just double check what else we can find at /webnotes. This is a snippet of what is found:

![webnotes](https://imgur.com/S3bqGZK.png)

A username stinky@DeRPnStiNK is mentioned. This is worth noting. There was also a mention of a robots.txt file in the nmap scan so let's have a look at that as well:

![robots](https://imgur.com/0Co5GVQ.png)

Paths /php and /temporary are listed in this. Let's check them out:

![phptemp](https://imgur.com/NaQDqrv.png)

This seems to be a rabbit hole. Nothing to see here. Now let's continue to updating our hosts file.

A hosts file is a plain text file that all operating systems use to translate hostnames into IP addresses. You can read more about this [here](https://vitux.com/linux-hosts-file/).

Edit the /etc/hosts file with gedit or nano.

<pre>nano /etc/hosts</pre>

Add the machines IP address to a hostname.

<pre>192.168.56.103 derpnstink.local</pre>

This should now be accessible. You can visit the webpage and enumerate this further.

Use dirb (Web Content Scanner) to find more paths to this website to hopefully find the blog the users have been talking about.

<pre>dirb http://derpnstink.local/</pre>

![dirb](https://imgur.com/4jMsWGK.png)

Visit the link and there is a blog. According to the dirb results, there seems to be WordPress running. This is a snippet of my results:

![dirbsnip](https://imgur.com/SBIVNDJ.png)

Wordpress is one of the most popular website making tools which attracts lots of hackers. Let's use the popular Wordpress Vulnerability scanner WPScan. Here is a link to their [website](https://github.com/wpscanteam/wpscan). 

Kali Linux should already have this installed so I will be executing the following command:

<pre>wpscan --url http://derpnstink.local/weblog/ -e u,vp,vt</pre>

-e refers to the enumeration option. u,vp and vt means the scan will be looking for users, vulnerable plugins and vulnerable themes.

![wpscan](https://imgur.com/BpqnfvM.png)

Some users have evidently been found: unclestinky and admin.

All the exploits given by WPScan seem to be authenticated which is not yet possible so we will look for a way to gain user access. 

I will try bruteforcing since WPScan has an option for bruting passwords. This is the command:

<pre>wpscan --url http://derpnstink.local/weblog/ --passwords /usr/share/wordlists/rockyou.txt --username < user > </pre>

I have used the --passwords option to indicate a brute forcing session. I have given a path to a passwords list and will provide a username after the --username parameter. I first tried unclestinky the admin.

After some time there was no results. I tried the default admin credentials which gave a successful login. For some reason the rockyou.txt file didn't contain "admin".  
