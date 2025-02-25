+++
title = 'Bounty Hacker: TryHackMe Writeup'
date = "2020-09-05"
draft = false
+++

You talked a big game about being the most elite hacker in the solar system. Prove it and claim your right to the status of Elite Bounty Hacker!
 
---
## Recon 
After we have deployed the room let's take the IP address and plug it into our web browser and see if anything comes up.
It looks like we have a hit.
![webService](/images/BountyHacker/1.png "webService")  
Ok it looks like the maker of this room is a Cowboy BeBop fan. How about we check out the source code to see if there are any hints as to `Question #3`.  

![source](/images/BountyHacker/2.png "source")  
There does not seem to be anything useful here... So let's move on to nmap and see it we can find anything else.  

---
## Sacnning & Enumeration 

NMAP: `nmap -sC -sV -T4 -oN nmap/result <$ip>`

Breakdown:
```
-A: do an extensive scan on these ports

-sC: Scan with default NSE scripts. Considered useful for discovery and safe

-sV: Attempts to determine the version of the service running on port

-T4: speed of nmap scan is 4/5 (personal preference of mine)

-oN: Normal output to the file normal.file
```  
![nmap](/images/BountyHacker/3.png "nmap")  
Once I have my results of my scan the first port catches my attention.  
It looks like there is an FTP service running and it allows `anonymous` logons.  
Lets try and connect to this FTP.  
`ftp <$ip>`  
The username is going to be anonymous.  
Once I am logged in I checked to see what files where available.
![ftp](/images/BountyHacker/4.png "ftp")  
It looks like there are two files for us. I am going to use the get command to download each of those files to my local computer.  
Now that they are downloaded I can cat the content of task.txt. It seems like I now know who created the task list.. and just like that I now have the answer to `Question #3`  

![task](/images/BountyHacker/5.png "task")  

It is time to move on to `Question #4`. What service can you bruteforce with the text file found?  
The text file that TryHackMe is refering to is the locks.txt. When you examine the content of it you will see that it looks a lot like a password list.  
Also if we refer back to our nmap scan there was only one service that really required a password and that was SSH. How about we try that and see... Congratulations you just found the answer to `Question #4`.  

Now I am going to use the locks.txt file and the name lin in combonation with Hydra to see if we are able to brute force any passwords.  

Hydra: `hydra -l lin -P locks.txt ssh://<$ip>`  
![hydra](/images/BountyHacker/6.png "hydra")  
It looks like this successful, now it is time to try and SSH in with our newly found credentials.

SSH: `ssh lin@<ip>`  

![creds](/images/BountyHacker/7.png "creds")  
As soon as I am connected, let's start checked to see if we can fine `user.txt` file mentioned in `Question #6`.
Right away the user.txt file can be found our present working directory.  
 ![user](/images/BountyHacker/8.png "user")  

---
## Privilege Escalation
 
 Now it is time for the fun to begin, we have compromised this box but we need to elavate out privileges from a stander user to a root users.  
 What we can do here is check and see what commands can be ran with sudo.  
 ![sudo](/images/BountyHacker/9.png "sudo")  
 Seems like I am able to run `/bin/tar` with sudo.  
After checking Google and see if there are any know exploites. I came [gtfobins](https://gtfobins.github.io/gtfobins/tar/) and it has a nice one-liner that we can use.

Command: `sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh`  
After running this one-liner it drops down into a root shell.  
![shell](/images/BountyHacker/10.png "shell")  
Now that it has been confirmed that our shell is root let's search for our root.txt file and the answer to `Question #7`.  
![root](/images/BountyHacker/11.png "root")  

---
## Congratulations you have just completed the Bounty Hacker from TryHackMe challenge!

