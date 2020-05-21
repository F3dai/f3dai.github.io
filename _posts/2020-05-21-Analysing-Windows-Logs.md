---
published: true
title: Analysing Windows Logs
category: other
image: 'https://imgur.com/ivkrOwm.png'
author: F3dai
---

Analysing log files is extremely common when working in cybersecurity. Keeping and managing secure systems means you will have to deal with log files. This article will be discussing the use of a script that I have created as a university assignment.

Some of the common reasons why log analysis is performed could be:

- Compliance with security policies
- Compliance with audit or regulation
- System troubleshooting
- Forensics (during investigations or in response to subpoena)
- Security incident response
- Understanding online user behavior

Thanks to scripting languages like Python, mass log analysis can be made extremely easy. 

### EVTX log files

.evtx files are log files created by the Windows 7 Event Viewer; contains a list of events recorded by Windows; saved in a proprietary binary format that can only be viewed within the Event Viewer program. 

You can read more about these files [here](https://fileinfo.com/extension/evtx)

### Getting Started

Head over to my Github repository with the following link:

[F3dai Github](https://github.com/F3dai/Windows-Log-Analyser)

Here you can find all the scripts needed for this analysis. There is also an archived .7z file with a set of .evtx log files for example. You will need all these files.

As the README.md explains, this is a set of python scripts to analyse, present and visualise Windows .evtx log files.

You will need:

- Python3
- Evtx python module

You may install the python module with the following command:

<pre>pip3 install python-evtx</pre>

Unzip all the evtx files. This is what the compilation of files should look like:

![ls](https://imgur.com/PTvSnrW.png)

Here is an explanation of all the different python files:

- convert.py - Check and clear any existing xml files before converting .evtx event log files to .xml files for analysis.

- analyse.py - Search for specific event IDs in any .xml files found. Please note: you will not be able to run this without any existing .xml files.

- visualise.py - Visualise a log file from analyse.py. Please note: you will not be able to run this without any analyse.py logs.

- Event ID information - Search for IDs or terms for descriptions of events. Ref: [ultimatewindowssecurity](www.ultimatewindowssecurity.com/securitylog/encyclopedia/)

- View Logs - View any log files created from convert.py, analyse.py and visualise.py without having to come out of the script. There is a plaintext datasheet file in the main directory that belongs to this script. This contains all the relevant security logs with descriptions.

- evtx_dump.py - Used for converting evtx to xml. [credit](https://github.com/williballenthin/python-evtx)

### Analysing Windows .evtx files

When everything is ready, run the hello.py script with the following command (As previously mentioned, it must be run with **python3**):

<pre>python3 hello.py</pre>

![py hello](https://imgur.com/UFyr2Nc.png)

Here is the main menu where we have an overview of all the different functions. The main 3 scripts are convert.py, analyse.py and visualise.py which is advised to be executed in it's respected order as analyse and visualise rely on previous script logs.

I will commence with the first, recomended step: convert.py.

**Convert**

![convert.py](https://imgur.com/3TZRmfr.png)

It has listed all known directories in the evtx logs folder. If any xml files exist, it will prompt the user to delete them all before starting the conversion. Please note, the user must have no xml files in order to convert.

Commence the conversion:

![conversion](https://imgur.com/eV5UUR3.png)

All evtx files have now been converted to xml files, ready for analysis!

**Analyse**

Let's analyse these xml files now by selecting 2 on the main menu:

![analyse options](https://imgur.com/3G2Ndd6.png)




