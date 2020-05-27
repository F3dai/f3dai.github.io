---
published: true
category: Ethical
author: F3dai
title: Command Injection / Execution (DVWA)
image: 'https://imgur.com/sUEKMTF.png'
---

This post is part of the DVWA series. You can find more DVWA walkthrough's in the [Ethical Hacking](/ethicalhacking/) section of this website. According to [OWASP](https://owasp.org/www-community/attacks/Code_Injection), Code Injection is the general term for attack types which consist of injecting code that is then interpreted/executed by the application. 

Attackers often look for poor handling of untrusted data. These exploits are possible as a result of a lack of proper input/output data validation.

This article will be focusing on the exploitation of the intentionally vulnerable web application DVWA. You can more information about this on their website:

[Damn Vulnerable Web Application](http://www.dvwa.co.uk/)

## Low security

Change the setting to low security on DVWA:

![low setting](https://imgur.com/xxzUmYh.png)

Go to the Command Execution page on DVWA. If you are not logged in, these are the credentials:

<pre>admin
password</pre>

Having a quick look at the source code provided, we can identify an unsanitised / un-filtered command execution. 

<pre> &lt;?php

if( isset( $_POST[ 'submit' ] ) ) {

    $target = $_REQUEST[ 'ip' ];

    // Determine OS and execute the ping command.
    if (stristr(php_uname('s'), 'Windows NT')) { 
    
        $cmd = shell_exec( 'ping  ' . $target );
        echo '&lt;pre&gt;'.$cmd.'&lt;/pre&gt;';
        
    } else { 
    
        $cmd = shell_exec( 'ping  -c 3 ' . $target );
        echo '&lt;pre&gt;'.$cmd.'&lt;/pre&gt;';
        
    }
    
}
?&gt; </pre>

The page essentially takes the IP given and attempts to ping it with the following code:

<pre>shell_exec( 'ping  ' . $target ); </pre>

The code does not check if $target matches an IP Address, so we can exploit this. When executing shell commands, we can actually execute multiple commands in one line by using the following:

";" - Determines end of the command
"&&" - AND operator
"|" - PIPE operator

So based on this information, we can specify any IP with ";" at the end and add our command to execute. Trying the following command gives the following result:

<pre>8.8.8.8; pwd</pre>

![exec low](https://imgur.com/CPlhicJ.png)

As you can see, the "pwd" command to show the current path has worked. This was we utilise the arbitrary code execution to gain further access to the system or see any sensitive information (such as the /etc/passwd file). This would be the executed command on the system:

<pre>ping -c 4 8.8.8.8; pwd</pre>

## Medium Security

Now let's focus on the "medium" security setting for the same section. This is the source code provided:

<pre></pre>

The page has added some minimal filtering to exclude exclude "&&" and ";". As noted earlier, we can use the PIPE operator. Adding an IP with a PIPE operator with our own command will result in the following:

<pre>8.8.8.8 | pwd</pre>

![exec med](https://imgur.com/xhCAbRJ.png)

Although there is some improvement, it is still vulnerable.

## High Security

Here is the source code for the command execution page on the high-security setting:

<pre>&lt;?php

if( isset( $_POST[ 'submit'] ) ) {

    $target = $_REQUEST[ 'ip' ];

    // Remove any of the charactars in the array (blacklist).
    $substitutions = array(
        '&amp;&amp;' =&gt; '',
        ';' =&gt; '',
    );

    $target = str_replace( array_keys( $substitutions ), $substitutions, $target );
    
    // Determine OS and execute the ping command.
    if (stristr(php_uname('s'), 'Windows NT')) { 
    
        $cmd = shell_exec( 'ping  ' . $target );
        echo '&lt;pre&gt;'.$cmd.'&lt;/pre&gt;';
        
    } else { 
    
        $cmd = shell_exec( 'ping  -c 3 ' . $target );
        echo '&lt;pre&gt;'.$cmd.'&lt;/pre&gt;';
        
    }
}

?&gt;  </pre>

Now the source code checks if each octet of the given IP is a valid IP by diving the IP into octets and checking if they are integers. This means any extra command we include like before will most likely not work as they aren't in an IP format.

For this reason, the system has made command injection very hard. Please note, depending on which DVWA version you are using, the source code may vary. This is considered the "impossible" level. 

**Further Exploitation:**

You can refer to this page for useful one-liners which can be used to gain proper access or a "reverse shell" using a vulnerability like this:

[Reverse Shell Cheatsheet](cheatsheet/Reverse_Payload_Cheatsheet/)



