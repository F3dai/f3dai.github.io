---
published: true
category: Ethical
image: 'https://imgur.com/Qcm0adH.png'
title: File Inclusion (DVWA)
author: F3dai
---
This post is part of the DVWA series. You can find more DVWA walkthrough's in the [Ethical Hacking](/ethicalhacking/) section of this website. According to [OWASP](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.1-Testing_for_Local_File_Inclusion), LFI vulnerabilities allow for attackers to include a file. The vulnerability occurs due to the use of user-supplied input without proper validation.

The exploitation of this vulnerability may lead to the following:

- Code execution on the web server
- Code execution on the client-side such as JavaScript which can lead to other attacks such as cross site scripting (XSS)
- Denial of Service (DoS)
- Sensitive Information Disclosure

This article will be focusing on the exploitation of the intentionally vulnerable web application DVWA. You can more information about this on their website:

[Damn Vulnerable Web Application](http://www.dvwa.co.uk/)

*** Nothing contained in this article is intended to teach or encourage the use of security tools or methodologies for illegal or unethical purposes. Always act in a responsible manner. Make sure you have written permission from the proper individuals before you use any of the tools or techniques described here.

## Low Security

This is the source code provided for low security:

<pre>&lt;?php

    $file = $_GET['page']; //The page we wish to display 

?&gt; </pre>

The low and sometimes medium security settings on DVWA often have no input validation. This is the case for LFI low security. The code just gets any page we specify.

The first vulnerability that comes to mind is path traversal. As a result of poorly written code with no validation, we can construct relative directory traversal, attempting to see any files on the system. 

According to [OWASP](https://owasp.org/www-community/attacks/Path_Traversal), attackers that exploit this attempt to access files and directories that are stored outside the web root folder. 

To come out of the web root directory, we can use the dot dot slash notation to tell the browser to view files outside of the current path. (../../../). Let's attempt to view the /etc/passwd file.

Take the url of the page:

<pre>/dvwa/vulnerabilities/fi/?page=include.php</pre>

We can tell the web page to display any file after the "?page=". 

<pre>http://192.168.56.118/dvwa/vulnerabilities/fi/?page=../../../../../../etc/passwd</pre>

Or you could alternatively do:

<pre>http://192.168.56.118/dvwa/vulnerabilities/fi/?page=/etc/passwd</pre>

The above command would essentially go to /etc which is 6 directories "above" the current directory.

This is the result:

![low etc passwd](https://imgur.com/h69gy4m.png)

We are now viewing the sensitive information about all the users on the system. For reference, the /etc/passwd file contains the attributes of (i.e., basic information about) each user or account on a computer running Linux or another Unix-like operating system.

For this LFI attack, we can even include other websites in the get page code:

Not only can you view potentially sensitive information but you can also include your own malicious php files.

## Medium Security

Here is the source code for medium security:

<pre> &lt;?php

    $file = $_GET['page']; // The page we wish to display 

    // Bad input validation
    $file = str_replace(&quot;http://&quot;, &quot;&quot;, $file);
    $file = str_replace(&quot;https://&quot;, &quot;&quot;, $file);        


?&gt; </pre>

This has introduced some low level validation which replaces "https://" with nothing, attempting to avoid any external pages. Although DVWA has put external pages off in the web configuration, a way to bypass this would be to use uppercase letters such as "HtTpS://", or hthttp://tp:// => http://.

The directory traversal vulnerability still remains as demonstrated below. 

## High Security

Finally, the source code for the highest security setting:

<pre>&lt;?php
        
    $file = $_GET['page']; //The page we wish to display 

    // Only allow include.php
    if ( $file != &quot;include.php&quot; ) {
        echo &quot;ERROR: File not found!&quot;;
        exit;
    }
        
?&gt;  </pre>

As the code suggests, the page only gets the "include.php" page, else, display an error. This has significantly improved the sanitisation of file inclusion. I was unsure how to bypass this.

## Further Exploitation

The above methods only identify that there is a vulnerability. Attackers often use this to leverage their access to the system. This exploit can be used to gain a further foothold to "owning" the system.

For example, using the FI vulnerability, an attacker may inject some code to gain a reverse shell.

Depending on the system, the following cheat sheet may be useful when trying to using command injection on a vulnerability like LI:

[Reverse Shell Cheatsheet](/cheatsheet/Reverse_Payload_Cheatsheet/)

