---
published: false
---

This post is part of the DVWA series. You can find more DVWA walkthrough's in the [Ethical Hacking](/ethicalhacking/) section of this website. According to [OWASP](https://owasp.org/www-community/attacks/Code_Injection), Code Injection is the general term for attack types which consist of injecting code that is then interpreted/executed by the application. 

Attackers can utlisie different types of password cracking techniques which is elaborated more on this article:

[Password Cracking](/ethical/PasswordCracking/)

This article will be focusing on the exploitation of the intentionally vulnerable web application DVWA. You can more information about this on their website:

[Damn Vulnerable Web Application](http://www.dvwa.co.uk/)

*** Nothing contained in this article is intended to teach or encourage the use of security tools or methodologies for illegal or unethical purposes. Always act in a responsible manner. Make sure you have written permission from the proper individuals before you use any of the tools or techniques described here.

## Low security

Change the setting to low security on DVWA:

![low setting](https://imgur.com/xxzUmYh.png)

Go to the Brute Force page on DVWA. If you are not logged in, these are the credentials:

<pre>admin
password</pre>

Do not log into the login page on the brute forcing section. We are presented with a username input and password input. Our goal is to gain authorised access to the log in page by utlising the Brute Force technique with a dictionary attack.

A dictionary attack essentially systematically uses words from a list as passwords to gain access to a system.

Let's set up our attack with the following steps. I will be performing this attack on a Kali Linux machine.

**Proxy**

We will be using Burp Suite to intercept the request from us to DVWA. Set up a proxy, I am using Firefox. Go to your browser menu > preferences > Network Settings > (a box should pop up) select manual configuration and enter the localhost address "127.0.0.1" as your HTTP proxy > clear the "No proxy for" box and ok the settings.

![proxy](https://imgur.com/9GLpaVL.png)

**Burp Suite**

Open Burp Suite on your system. This will be used to analyse the web request.

![open burp](https://imgur.com/2ZZ2IXY.png)

There are many tools on Burp Suite, however, we are only interested in the intercepted request. Go to the Proxy tab. The intercept should be on. 

![proxy tab](https://imgur.com/FMz7VO7.png)

**Intercept**

Now we are ready to analyse the request, go back to the log in page on DVWA and enter any random credentials. It does not matter what is entered, as long as you provide incorrect details.

![test creds](https://imgur.com/mzWzbDu.png)

Once you submit the details, the request should pop up in Burp Suite. 

This is my request to http://192.168.56.118:

<pre>GET /dvwa/vulnerabilities/brute/?username=test&password=test&Login=Login HTTP/1.1
Host: 192.168.56.118
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://192.168.56.118/dvwa/vulnerabilities/brute/
Connection: close
Cookie: security=low; PHPSESSID=3f319f0402f5ec006e7b70da8ea92bf4
Upgrade-Insecure-Requests: 1</pre>

If you are unaware, there are different types of web requests such as GET, POST, PUT and more. Please read more about this on this website if you want to research more:

[Mozilla Developer](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods)

Here we can see all types of information being sent to DVWA such as the GET request, host, browser info, cookie etc. Right now we are interested in the GET request and cookie. Take note of these as we will use this repeatly for the brute forcing. 

<pre>GET /dvwa/vulnerabilities/brute/?username=test&password=test&Login=Login HTTP/1.1
Cookie: security=low; PHPSESSID=3f319f0402f5ec006e7b70da8ea92bf4</pre>

Once you have this information, you can close down Burp Suite and turn your proxy off. 

**Hydra**

Now we are ready to construct our Hydra command. [THC-Hydra](https://tools.kali.org/password-attacks/hydra) is a tool which is used for login cracking. Here is the template we will use:

<pre>hydra [host] -l [user] -P [passwordlist] http-get-form “ [username ^USER^ password ^PASS^ ] : F=[Failed Message] : H=Cookie: [Cookie Value]” </pre>

So here is the command that I need to use:

<pre> hydra 192.168.56.118 -l admin -P "/usr/share/wordlists/rockyou.txt" http-get-form "/dvwa/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:F=Username and/or password incorrect.:H=Cookie: PHPSESSID=3f319f0402f5ec006e7b70da8ea92bf4"</pre>

This command is first specifying our host IP, -l user, -P wordlist, http-get-form is the type of traffic (this is a GET request). "/login/page : username and password field : F = Our message we noted if login is incorrect : H=Cookie: Our cookie from this session". Run this command on your machine, you can include -V as well to see the verbose output to see what's going on.

I have used a premade wordlist in Kali Linux, but you can use anything you like. This is the output: 

![dictionary](https://imgur.com/G1gxpfG.png) 

The password “password” has been found by looking at the responses of the web requests made by hydra. Hydra keeps testing passwords until it receives anything but the failed login message we specified.

We can confirm this with the DVWA page. 

![success](https://imgur.com/alQco9v.png)

## Medium Security

Let's start by running our previous hydra command from low security.

If you run the command again, you will notice the attack taking a significantly longer amount of time. Let's see how long the response time is. 













