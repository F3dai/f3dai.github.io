---
published: true
title: Retro - TryHackMe Walkthrough
category: Writeup
author: F3dai
image: 'https://imgur.com/RIQkBU2'
---
Retro - "New high score! Can you time travel? If not, you might want to think about the next best thing." This is a TryHackMe box. To access this you must sign up to [https://tryhackme.com/](https://tryhackme.com/). 

**URL:** [Retro](https://tryhackme.com/room/retro)

**Difficulty:** Hard

**Author:** DarkStar7471

We are given the IP 10.10.120.163, add it to /etc/hosts and run a portscan:

<pre>nmap -p- -A retro.thm -o portscan</pre>

<pre>PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: RETROWEB
|   NetBIOS_Domain_Name: RETROWEB
|   NetBIOS_Computer_Name: RETROWEB
|   DNS_Domain_Name: RetroWeb
|   DNS_Computer_Name: RetroWeb
|   Product_Version: 10.0.14393
|_  System_Time: 2020-07-29T15:13:43+00:00
| ssl-cert: Subject: commonName=RetroWeb
| Not valid before: 2020-07-28T15:04:04
|_Not valid after:  2021-01-27T15:04:04
|_ssl-date: 2020-07-29T15:13:44+00:00; +1s from scanner time.
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
</pre>

Check out port 80

![port 80](https://imgur.com/eUbzOuP.png)

## What is the hidden directory which the website lives on?

Let's run a dirbuster scan. The normal wordlist didn't get any results so I used directory-list-2.3-medium.txt

Wordlist:

<pre>/usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt</pre>

Results:

<pre>https://imgur.com/fyzqHrt.png</pre>

We found a directory called *Retro*.

<pre>http://retro.thm/retro/</pre>

![Retro directory](https://imgur.com/nccw0E4.png)

## user.txt

We have more to enumerate. I saw some Wordpress related directories on our dirbuster scan so let's visit *wp-admin* to confirm.

The URL redirects us to 

<pre>https://localhost/retro/wp-login.php?redirect_to=http%3A%2F%2Fretro.thm%2Fretro%2Fwp-admin%2F&reauth=1</pre>

Let's add localhost to our hosts file:

<pre>sudo nano /etc/hosts
10.10.120.163 retro.thm localhost</pre>

Revisit the wp-admin page:

![wp-admin](https://imgur.com/mxBpuMF.png)

Let's run WPScan:

<pre>wpscan --url http://retro.thm/retro -e u</pre>

This will scan the website and find possible users.

<pre>[i] User(s) Identified:

[+] wade
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://retro.thm/retro/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] Wade
 | Found By: Rss Generator (Passive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)</pre>
 
 We've found Wade. The first result says it was found by Author posts so I began studying their content. 
 
 I saw that there was a comment made by Waze on one of his posts:
 
 ![Comment side](https://imgur.com/pERGsIH.png)
 
 Here is the comment:
 
 ![Comment](https://imgur.com/5N1Ib5w.png)
 
 <pre>Leaving myself a note here just in case I forget how to spell it: parzival</pre>
 
 Turns out this is the password for his account. Log into his account:
 
 ![wp admin login](https://imgur.com/EqI6Bfx.png)
 
 I looked for a bit but this seems to be a dead end. Looked up all the installed plugins, Wordpress version etc but no obviously vulnerabilities. 
 
 What else could we do with these credentials? Maybe the other open port for RDP. Let's try and connect using the following command:
 
 <pre>xfreerdp /u:wade /p:parzival /v:retro.thm</pre>
 
 We successfully got an RDP session:
 
 ![RDP](https://imgur.com/EEEpRetro CTF

Start with portscan:

<pre>nmap -p- -A retro.thm -o portscan</pre>

<pre>PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: RETROWEB
|   NetBIOS_Domain_Name: RETROWEB
|   NetBIOS_Computer_Name: RETROWEB
|   DNS_Domain_Name: RetroWeb
|   DNS_Computer_Name: RetroWeb
|   Product_Version: 10.0.14393
|_  System_Time: 2020-07-29T15:13:43+00:00
| ssl-cert: Subject: commonName=RetroWeb
| Not valid before: 2020-07-28T15:04:04
|_Not valid after:  2021-01-27T15:04:04
|_ssl-date: 2020-07-29T15:13:44+00:00; +1s from scanner time.
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
</pre>

Check out port 80

![port 80](https://imgur.com/eUbzOuP.png)

## What is the hidden directory which the website lives on?

Let's run a dirbuster scan. The normal wordlist didn't get any results so I used directory-list-2.3-medium.txt

Wordlist:

<pre>/usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt</pre>

Results:

<pre>https://imgur.com/fyzqHrt.png</pre>

We found a directory called *Retro*.

<pre>http://retro.thm/retro/</pre>

![Retro directory](https://imgur.com/nccw0E4.png)

## user.txt

We have more to enumerate. I saw some Wordpress related directories on our dirbuster scan so let's visit *wp-admin* to confirm.

The URL redirects us to 

<pre>https://localhost/retro/wp-login.php?redirect_to=http%3A%2F%2Fretro.thm%2Fretro%2Fwp-admin%2F&reauth=1</pre>

Let's add localhost to our hosts file:

<pre>sudo nano /etc/hosts
10.10.120.163 retro.thm localhost</pre>

Revisit the wp-admin page:

![wp-admin](https://imgur.com/mxBpuMF.png)

Let's run WPScan:

<pre>wpscan --url http://retro.thm/retro -e u</pre>

This will scan the website and find possible users.

<pre>[i] User(s) Identified:

[+] wade
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://retro.thm/retro/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] Wade
 | Found By: Rss Generator (Passive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)</pre>
 
 We've found Wade. The first result says it was found by Author posts so I began studying their content. 
 
 I saw that there was a comment made by Waze on one of his posts:
 
 ![Comment side](https://imgur.com/pERGsIH.png)
 
 Here is the comment:
 
 ![Comment](https://imgur.com/5N1Ib5w.png)
 
 <pre>Leaving myself a note here just in case I forget how to spell it: parzival</pre>
 
 Turns out this is the password for his account. Log into his account:
 
 ![wp admin login](https://imgur.com/EqI6Bfx.png)
 
 I looked for a bit but this seems to be a dead end. Looked up all the installed plugins, Wordpress version etc but no obviously vulnerabilities. 
 
 What else could we do with these credentials? Maybe the other open port for RDP. Let's try and connect using the following command:
 
 <pre>xfreerdp /u:wade /p:parzival /v:retro.thm</pre>
 
 We successfully got an RDP session:
 
 ![RDP](https://imgur.com/EEEpXil.png)
 
 The user flag is on the desktop.
 
 ## root.txt
 
I saw the machine has Chrome installed which is not my default so I checked to see if I could find anything on there. 

I saw this:

![Bookmarked](https://imgur.com/4zjBV6L.png)

<pre>https://nvd.nist.gov/vuln/detail/CVE-2019-1388</pre>

I found this exploit:

[CVE-2019-1388 - Github](https://github.com/jas502n/CVE-2019-1388)

Let's download the .exe file to our local machine and set up an HTTP server so we can transfer it to the Windows machine.

For simplicity, I visited http://10.9.6.63:8080/ on Chrome and installed the binary.

I followed the steps but this didn't lead me anywhere. Some further inspection of the Windows system:

<pre>C:\Users\Wade\Documents>systeminfo

Host Name:                 RETROWEB
OS Name:                   Microsoft Windows Server 2016 Standard
OS Version:                10.0.14393 N/A Build 14393
OS Manufacturer:           Microsoft Corporation</pre>

Researching OS 10.0.14393 N/A Build 14393

I also ran Windows Exploit Suggester:

[Windows Exploit Suggester - Github](https://github.com/AonCyberLabs/Windows-Exploit-Suggester)

And I found this exploit:

[CVE-2017-0213 - Github](https://github.com/SecWiki/windows-kernel-exploits/tree/master/CVE-2017-0213)

So all we need to do is install CVE-2017-0213_x64.exe and run it on our target system. 

![Binary 2](https://imgur.com/Z16TSyC.png)

Now run the executable (not from PowerShell) and you will be given a terminal with elevated privileges.

![rooted](https://imgur.com/bl6ZRVP.png)

The last flag is in the admins Desktop directory.
Xil.png)
 
 The user flag is on the desktop.
 
 ## root.txt
 
I saw the machine has Chrome installed which is not my default so I checked to see if I could find anything on there. 

I saw this:

![Bookmarked](https://imgur.com/4zjBV6L.png)

<pre>https://nvd.nist.gov/vuln/detail/CVE-2019-1388</pre>

I found this exploit:

[CVE-2019-1388 - Github](https://github.com/jas502n/CVE-2019-1388)

Let's download the .exe file to our local machine and set up an HTTP server so we can transfer it to the Windows machine.

For simplicity, I visited http://10.9.6.63:8080/ on Chrome and installed the binary.

I followed the steps but this didn't lead me anywhere. Some further inspection of the Windows system:

<pre>C:\Users\Wade\Documents>systeminfo

Host Name:                 RETROWEB
OS Name:                   Microsoft Windows Server 2016 Standard
OS Version:                10.0.14393 N/A Build 14393
OS Manufacturer:           Microsoft Corporation</pre>

Researching OS 10.0.14393 N/A Build 14393

I also ran Windows Exploit Suggester:

[Windows Exploit Suggester - Github](https://github.com/AonCyberLabs/Windows-Exploit-Suggester)

And I found this exploit:

[CVE-2017-0213 - Github](https://github.com/SecWiki/windows-kernel-exploits/tree/master/CVE-2017-0213)

So all we need to do is install CVE-2017-0213_x64.exe and run it on our target system. 

![Binary 2](https://imgur.com/Z16TSyC.png)

Now run the executable (not from PowerShell) and you will be given a terminal with elevated privileges.

![rooted](https://imgur.com/bl6ZRVP.png)

The last flag is in the admins Desktop directory.

