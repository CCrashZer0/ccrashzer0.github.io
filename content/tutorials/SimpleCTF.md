+++
title = 'Simple CTF: TryHackMe Writeup'
date = "2020-08-06"
draft = false
+++

# Try Hack Me - Simple CTF
Description: Beginner level ctf

---
## Objectives
1. How many services are running under port 1000?
2. What is running on the higher port?
3. What's the CVE you're using against the application?
4. To what kind of vulnerability is the application vulnerable?
5. What's the password?
6. Where can you login with the details obtained?
7. What's the user flag?
8. Is there any other user in the home directory? What's its name?
9. What can you leverage to spawn a privileged shell?
10. What's the root flag?
---
## Recon
Once the machine has been deployed we are going to take the IP address and plug it into our web browser to see if their is a webservice running on it.  
{{< figure src="/images/SimpleCTF/webService.png">}}

It seems that we do have a web service running but all we get is your stander Apache default screen.

---
## Scanning, and Enumeration

Now since one of our objectives it to find what is running on the higher port numbers we are going to scan all ports with the `-p-` switch.  

Command: `nmap -A -T4 -sC -sV -p- -Pn -oN nmap/results.txt <ip>`
Command Breakdown:
```
-A: do an extensive scan on these ports

-T4: speed of nmap scan is 4/5 (personal preference of mine)

-sC: Scan with default NSE scripts. Considered useful for discovery and safe

-sV: Attempts to determine the version of the service running on port

-Pn: Disable host discovery. Port scan only.

-oN: Normal output to the file normal.file
```
{{< figure src="/images/SimpleCTF/nmapResults.png">}}

Now that I have the nmap results we can see that there are three ports that are currently open.  
Two of them are under the number of 1000 and one is above 1000.  
So that allows us to answer both objects one and two.  

Now I will need to try and locate the CVE that we are going to use to compromise this box with.  
In order to find the CVE that we are going to be using in this challenge I feel like I am going to need to do some more recon and look for additional directories.  
This is where I fire up gobuster along with our `common.txt` file. This will search for you guessed it.. Common directory names. 

Command: `gbuster dir - u http://<Maching_IP> -t 40 -w /usr/shared/`  
Command Breakdown:
```
-u: Full URL (including scheme), or base domain name.

-t: Thread Usage, declare how mand thread you would like gobuster to use. (Default is 10)

-w: Path to the wordlist used for brute forcing (use – for stdin)

```

{{< figure src="/images/SimpleCTF/gbResults.png">}}

It seems thate we have same additions, more specifical we have a directory called `simple`. Now if I plug that directory into my browser I am presented with a nice website talking about CMS Made Simple.
At the bottom of this site I am given the version number. It seems that this CMS is running on version 2.2.8  

{{< figure src="/images/SimpleCTF/CMS.png">}}

Now that I know that I am dealing with a CMS plateform I am going to use `searchsploit` to see if there are any know vulnerabilities for it.  
The one that sticks out to me is a SQL Injection, it seems that it will work on CMS Made Simple version 2.2.10 and lower.  

{{< figure src="/images/SimpleCTF/searchsploit.png">}}
## Privilege escalation

Let's pull up Google and see if their is an CVE assocaited with the vulnerability.  

After a quick search using keywords such as `CMS Made Simple Exploite`. I was able to find something on exploite database and it looks like there is a CVE associate with it as well and just like that we have found the answer to object three and four.  

{{< figure src="/images/SimpleCTF/google.png">}}

To exploit this vulnerability I can use the python script that is available. When running the script for the first time you will encounter and error. In order to correct the error you are going to need to use pip to install the termcolor module.  
Command: `pip install termcolor`  

Now that we have all of our additional modules installed we can run this exploit.  
Notice that I am using a [SecList]('https://github.com/danielmiessler/SecLists'). This is not part of Kail and you will need to download it sepratly!  

Command: `python expolit.py -u http://MACHINE_IP --crack -w /usr/share/wordlists/SecLists/Passwords/Common-Credentials/best110.txt`

{{< figure src="/images/SimpleCTF/creds.png">}}

It seems that I have found login credentials and the answer to objective five.  
Now if I recall there was an SSH port open, maybe we can use what we just found to try and login. I was able to connect using SSH.  
This has also provided us with the answer to objective six!  

{{< figure src="/images/SimpleCTF/ssh.png">}}

Once I am connected via SSH the first thing I am going to do is use the `ls` command and see what shows up. Just like that I have foudn the user flag and completed objective seven. 

{{< figure src="/images/SimpleCTF/userFlag.png">}}

Objective eight states that there is another user on this challenge. So let's check the home directory and see who it is.  
{{< figure src="/images/SimpleCTF/secUser.png">}}

Now before I move any futher, I would like to check and see if I am able to run anything as root and without a password.  
{{< figure src="/images/SimpleCTF/noRoot.png">}}

Now it does not seem like it but I just found the answer to objective nine.

---
## Privilege Escalation
It looks like I am able to run vim as root. So the time has come for me to see if I can call a root shell from within vim.  
The first thing that I do is use vim to create a test file.  

Command: `sudo vim test`  
Once my test file is opend up I now need to try and gain a shell. This can be done by pressing `ctrl + :` and then entering in `!sh`.  
After running that command I am dropped into a root shell and with that access I am able to complete object ten!
{{< figure src="/images/SimpleCTF/rootFlag.png">}}

# Congratulations you have just completed the Simple CTF challenge!
