---
title = "Take Out The Trash - CTF Walkthrough"
date = 2025-09-01T14:34:50.223Z
draft = false
tags = [
    "blog",
]
---

Take out the trash is a boot2root style CTF challenge. Your objective here is to compromise a user account and then determine the best path to take in order to escalate your privileges to the root user and retrieve the root flag.

In order to discover our point of entrance we will first need to do a little bit of recon. This can be done by using the tool nmap. It comes standard in most pen testing distros. The command I like to use is `nmap -sC -sV -T4 -A <IP>`. Based on the results, I can see this server has an FTP service running. It allows anonymous login, which should be our entry point.
The nmap results reveal an FTP server running with anonymous login enabled our potential entry point. So this should be the foothold that we have been looking for

We will connect to the FTP using `ftp <IP ADDRESS>`. When prompted for a username, we'll use 'anonymous'. No password is needed as it logs us directly into the application. When I try to list the directory, I get a message saying "Entering Extended Passive Mode". I had never seen this before, so I researched the issue and found a solution. Simply typing `passive` disables passive mode. This allows you to use commands like `ls` to list the directory.

After successfully running `ls`, you can see two files. One is called "file.txt". The other appears to be a JPG image with a long string of letters and numbers as its name. Let's go ahead and download each of these files using the get command. We'll use `get file.txt` to retrieve the text file. We'll repeat the same command to download the JPG image.

Now that we have both files saved to our desktop, let's examine them for useful information. Upon opening the file.txt we see that it appears to be a note from the admin to a user by the name of CCrashZer0. So now we know of at least one user that is on this system.

Let's take a look at the second file that we got. It appears to be an image, but it has an unusual name. Notice the two equal signs at the end—this is called padding. You'll see this frequently with base64 encoded strings. If we use a tool like CyberChef we can easily decode this string. Once decoded, the string still appears scrambled with letters, numbers, and symbols. We'll return to this later, so note the decoded string for now.

Remember that image that we found. Let's take a closer look at it. We are going to use an application called Steghide. Using this application we can check and see if there was anything hidden inside it. We'll use the command `steghide extract -sf <ImageName>.jpg`. After running this, we're prompted for a passphrase. Let's try that random string we decoded earlier to see if it works as a password. Using that string successfully extracts a file called `list.txt`. Examining its contents reveals it's a password list.

 With the list that we just acquired and the fact that we know the name of the user thanks to the note we found earlier we can now try and brute force the password using the Hydra application. We'll run `hydra -l userName -P list.txt <IP Address> ssh -t 4`. Hydra will try each password in the list until it finds a match or exhausts all options. In our case we do find the password using the list that we find.

Now we can SSH into the target machine using `ssh userName@ipAddress`. After entering the password, we establish a connection. So lets see what we are able to find now. Running `ls` immediately shows a "user.txt" file. Let's read it with the `cat` command. Success! We now have the user flag.

Now we only have one flag let to find and that is the root flag. So the question becomes at this point how are we going to escalate our privileges. We need something that will allow us to become the root user. There are several ways to escalate privileges. The easiest is to start with `sudo -l`. This shows what we can run as sudo. We have access to two things. A script called "EmptyTrash.sh" and nano, how about we look at this script that we have access to. Close inspection reveals this script just empties the trash on the host machine. However, we also have access to the nano text editor. This means we could edit the script to include a reverse shell.

Let's head over to payload all the things and pick out a reverse shell. I am going to use the bash shell but you can use whatever you wanted to. Once we set up the commands and change the IP address to match our attacking machine, we need to set up a listener. On our attacker device, we'll use `nc -u -lvp <port>`.

Everything is set up. Now we need to execute our plan and see if the reverse shell connects to our listener. Back on the victim machine, let's navigate to the empty trash script's directory. Once there will execute the script with `./EmptyTrash.sh`. Looking back at our attacker machine we can see that our listener has caught something. When we run `whoami` we can see that the prompt comes back with root... Congratulation we have root access.

Now we need to find the root flag. Similar to when we were looking for the user flag we wanted to use the `ls` command to see what all we have around us and look at that there is the root.txt files that contains our flag. We are going to use `cat` again to read the flag and there it is.
