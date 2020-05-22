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
- Understanding online user behaviour

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

Here is the main menu where we have an overview of all the different functions. The main 3 scripts are convert.py, analyse.py and visualise.py which is advised to be executed in its respected order as analyse and visualise rely on previous script logs.

I will commence with the first, recommended step: convert.py.

**Convert**

![convert.py](https://imgur.com/3TZRmfr.png)

It has listed all known directories in the evtx logs folder. If any xml files exist, it will prompt the user to delete them all before starting the conversion. Please note, the user must have no xml files in order to convert.

Commence the conversion:

![conversion](https://imgur.com/eV5UUR3.png)

All evtx files have now been converted to xml files, ready for analysis!

**Analyse**

Let's analyse these xml files now by selecting 2 on the main menu:

![analyse options](https://imgur.com/3G2Ndd6.png)

We have 2 analysis options:

- Analysing the "Default IDs"
- Analysing "Specific IDs"

I have added default IDs to the script which can be considered important security-related events but you can also choose to search for your own IDs.

This section will effectively scan through all the converted evtx files and search for the IDs that have been specified.

I will be choosing the default ID option:

![analysed](https://imgur.com/q0Rj4qr.png)

This is a snippet of the end of the analysis. The script has successfully scanned through all Event IDs and checked whether they are any of the default IDs. You view the results in the log files, but we'll come to this soon. 

Now we are ready to visualise the results.

**Visualise**

I have selected the 3rd option on the main menu:

![visualise menu](https://imgur.com/wpGkvJq.png)

This section will now use the results from analyse.py to create a visualisation of event ID occurances. We are given the option to select which analyse result we want to use.

Let's use the results from what we just created, in my case, it's option (2).

![vis res](https://imgur.com/Xx0wCpr.png)

It has returned some results of the event ID count for all the IDs we specified (default IDs). If we press enter we can see these results in a graph:

![vis graph](https://imgur.com/BgYR4ht.png)

The event IDs, found on the y-axis have their corresponding count across the x-axis, as a horizontal bar chart.

This graph visualisation is extremely useful for understanding the kind of behaviour that was on the system.

**Event ID Information**

The next option in the main menu helps understand the returned results. Select number 4 from the menu:

![event db](https://imgur.com/Uee4IkA.png)

As the description says, this function of the script simply returns any result found from your query to further understand the event IDs. You can enter an Event ID number or some text to find the associated number. 

Let's search for some queries. We identified that events 4663 and 5156 were the most common IDs found from the results. Let's search these up to see what events mean:

![4663](https://imgur.com/s68x6iF.png)

We found that 4663 means "An attempt was made to access an object".

![5156](https://imgur.com/KJvGiKk.png)

We also found that 5156 means "The Filtering Platform has allowed a connection"

I will also search for the term "shut down" to see what the related event IDs are:

![shut down](https://imgur.com/i3TTD2U.png)

Useful information. We can also conveniently search all the default event IDs by inputting "/default".

**View Logs**

Finally, menu item 5 simply returns any of the log files we have created with our scripts through the terminal. No need to manually look at the files.  

![view logs](https://imgur.com/2tHOJA1.png)

For this example I will be looking at our Analyse results from earlier. I have selected the Analysis logs section:

![anyse logs](https://imgur.com/eGIldpo.png)

This is a list of all the results we have created. In my case, file number (2) is the file that we created for this article.

![view anyse](https://imgur.com/ALSTj4G.png)

Here we can scroll through what the terminal has returned and look at all the details from earlier. If you would like to access the whole file I suggest you manually look for the file in the following directory:

<pre>/CI5235_Logs</pre>

And that is my Windows Log Analyser script that I have made! Thank you for reading this article.

![thanks](https://imgur.com/bo97FSY.png)

You can also see my terminal recording of this tool being used here:

[![asciicast](https://asciinema.org/a/T0oJKJXoH3rVheQQGy7fFqe4P.svg)](https://asciinema.org/a/T0oJKJXoH3rVheQQGy7fFqe4P)


