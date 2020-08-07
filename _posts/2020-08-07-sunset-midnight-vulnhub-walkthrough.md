---
date: 2020-08-07 02:15:17
layout: post
title: "Sunset Midnight Vulnhub Walkthrough"
subtitle:
description: Sunset:Midnight is a boot2root machine, which is a part of sunset series available on Vulnhub.
image: /assets/img/uploads/sunset-midnight/midnight.webp
optimized_image: /assets/img/uploads/sunset-midnight/midnight-optimized.webp
category: walkthrough
tags: vulnhub boot2root 
author: yashsaxena
paginate: false
keywords: vulnhub, sunset midnight ctf, sunset midnight vulnhub walkthrough, midnight vulnhub, sunset midnight vulnhub, sunset midnight walkthrough, sunset midnight writeup, infosecarticles, infosec articles
---

Today I will be sharing a walkthrough of sunset:midnight, a boot2root machine which is available on Vulnhub. I really like all the machines from this author and I recommend you to try out his machines, these are actually good :)

## Initial Enumeration

As usual I started with nmap to check for open ports and services running in the system using the following command:<br>
<b>nmap -sC -sV -Pn -p- -T4 --max-rate=1000 192.168.1.182</b>

<img src="/assets/img/uploads/sunset-midnight/nmap.webp">

I started my enumeration from port 80 and found a website that is not reachable because we need to edit the host name in <b>/etc/hosts</b> file.

I again opened the website and confirmed that this website is developed using wordpress and without wasting my time, I performed a wpscan using the following command:<br>
<b>wpscan --url http://sunset-midnight --enumerate </b>

<img src="/assets/img/uploads/sunset-midnight/wpscan.webp">

I started to search about this plugin and found that it is vulnerable to SQL injection but I don't like SQL injection specially using the tool sqlmap, So I decided to do a bruteforce attack against mysql login with user root and using the dictionary <b>rockyou.txt.</b><br>
<b>hydra -l root -P ../rockyou.txt mysql://192.168.1.182</b>

<img src="/assets/img/uploads/sunset-midnight/hydra.webp">

Awesome! That's great. I found the password very quickly and now we can login into mysql as a root user.

<img src="/assets/img/uploads/sunset-midnight/mysql.webp">

First of all I decided to explore the mysql database and found an interesting table there.

<img src="/assets/img/uploads/sunset-midnight/database.webp">

Next, I dumped all the data inside the table <b>user.</b>

<img src="/assets/img/uploads/sunset-midnight/user.webp">

I found three hashes, and I got the password after cracking them with <b>John.</b> I was feeling so stupid at this stage because there is no point of cracking these hashes as I already know the root password, I am loosing my common sense. 

Okay, after this I decided to explore the database <b>wordpress_db</b> and found something interesting there (hash of user admin)

<img src="/assets/img/uploads/sunset-midnight/wp-users.webp">

I tried cracking it but failed. Next I thought we are in as user root and we can change the hash of user admin and will use that password to login into wordpress.

<img src="/assets/img/uploads/sunset-midnight/wp-passwd.webp">

Therefore after changing the credentials to admin:admin I was inside the Wordpress panel. Obtaining a low user shell from wordpress was not a difficult task, that's why I am not showing the steps. (You can still find them in some other walkthroughs shared on this blog.)

<img src="/assets/img/uploads/sunset-midnight/shell.webp">

After some enumeration I found the password of user <b>jose</b> and using that password I logged in as user <b>jose</b> using <b>ssh.</b>

<img src="/assets/img/uploads/sunset-midnight/ssh.webp">

## Root Shell

After that I started to search for SUID binaries and found an interesting one using the following command: <br>
<b>find / -perm -u=s -type f 2>/dev/null</b>

<img src="/assets/img/uploads/sunset-midnight/suid.webp">

Next, using the strings command on this binary I found that it uses the service command but without using the full path of the service command. So now we can change the PATH variable and will gain access to the root shell.

```r
cd /tmp
echo "/bin/sh" > service
chmod +x service
export PATH=/tmp/:$PATH
/usr/bin/status
```

<img src="/assets/img/uploads/sunset-midnight/root.webp">

That’s it! Thanks for reading. Stay tuned for similar walkthroughs and much more coming up in the near future!
If you have any queries, you can contact me <a href="/contact">here.</a>

NOTE: The awesome artwork used in this article was created by <a href="https://dribbble.com/studioMUTI">MUTI.</a>









