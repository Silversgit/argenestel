---
image: /assets/img/aster/aster.png
layout: post
title: Aster
date: 2020-10-6 00:00:01 +5:30
categories: [TryHackMe, Medium]
tags: [asterisk, java, uncomply6] # add tag
---

## Description:

The Following Post is writeup of Aster room of tryhackme <https://tryhackme.com/room/aster>

|Machine|Details
|:---|:--
|OS | Linux
|Rating | Medium
|Creator | stuxnet

## Summary

  The room have 5 ports open we can see asterisk service running 22,80,1720,2000,5038.
  Enumeration of

## Walkthrough

### Enumeration

Let's start with nmap scan.

```bash
argenestel@parrot  ~/Desktop/tryhackme/aster  mkdir nmap
argenestel@parrot  ~/Desktop/tryhackme/aster  rustscan 10.10.82.20
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
Open 10.10.82.20:22
Open 10.10.82.20:80
Open 10.10.82.20:1720
Open 10.10.82.20:2000
Open 10.10.82.20:5038
[~] Starting Nmap
[>] The Nmap command to be run is nmap -vvv -p 22,80,1720,2000,5038 10.10.82.20

Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-02 14:49 IST
Initiating Ping Scan at 14:49
Scanning 10.10.82.20 [2 ports]
Completed Ping Scan at 14:49, 0.33s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 14:49
Completed Parallel DNS resolution of 1 host. at 14:49, 0.05s elapsed
DNS resolution of 1 IPs took 0.05s. Mode: Async [#: 2, OK: 0, NX: 1, DR: 0, SF: 0, TR: 1, CN: 0]
Initiating Connect Scan at 14:49
Scanning 10.10.82.20 [5 ports]
Discovered open port 22/tcp on 10.10.82.20
Discovered open port 1720/tcp on 10.10.82.20
Discovered open port 80/tcp on 10.10.82.20
Discovered open port 2000/tcp on 10.10.82.20
Discovered open port 5038/tcp on 10.10.82.20
Completed Connect Scan at 14:49, 0.24s elapsed (5 total ports)
Nmap scan report for 10.10.82.20
Host is up, received syn-ack (0.29s latency).
Scanned at 2020-10-02 14:49:35 IST for 0s

PORT     STATE SERVICE    REASON
22/tcp   open  ssh        syn-ack
80/tcp   open  http       syn-ack
1720/tcp open  h323q931   syn-ack
2000/tcp open  cisco-sccp syn-ack
5038/tcp open  unknown    syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.74 seconds


PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 fe:e3:52:06:50:93:2e:3f:7a:aa:fc:69:dd:cd:14:a2 (RSA)
|   256 9c:4d:fd:a4:4e:18:ca:e2:c0:01:84:8c:d2:7a:51:f2 (ECDSA)
|_  256 c5:93:a6:0c:01:8a:68:63:d7:84:16:dc:2c:0a:96:1d (ED25519)
80/tcp   open  http        Apache httpd 2.4.18 ((Ubuntu))
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Aster CTF
1720/tcp open  h323q931?
2000/tcp open  cisco-sccp?
5038/tcp open  asterisk    Asterisk Call Manager 5.0.2
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Let's check port 80.

![port80](/assets/img/aster/port80.png)

#### Port 80

So there is pyc file we can use uncompyle6

```bash
✘ argenestel@parrot  ~/Desktop/tryhackme/aster  wget 10.10.82.20/output.pyc
--2020-10-02 15:10:00--  http://10.10.82.20/output.pyc
Connecting to 10.10.82.20:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1072 (1.0K) [application/x-python-code]
Saving to: ‘output.pyc’

output.pyc                        100%[==========================================================>]   1.05K  --.-KB/s    in 0s      

2020-10-02 15:10:01 (31.8 MB/s) - ‘output.pyc’ saved [1072/1072]

I saw the app let's check what it does

argenestel@parrot  ~/Desktop/tryhackme/aster  uncompyle6 output.pyc
# uncompyle6 version 3.7.4
# Python bytecode 2.7 (62211)
# Decompiled from: Python 3.8.5 (default, Aug  2 2020, 15:09:07)
# [GCC 10.2.0]
# Embedded file name: ./output.py
# Compiled at: 2020-08-11 12:29:35
import pyfiglet
o0OO00 = pyfiglet.figlet_format('Hello!!')
oO00oOo = '476f6f64206a6f622c2075736572202261646d696e2220746865206f70656e20736f75726365206672616d65776f726b20666f72206275696c64696e6720636f6d6d756e69636174696f6e732c20696e7374616c6c656420696e20746865207365727665722e'
OOOo0 = bytes.fromhex(oO00oOo)
Oooo000o = OOOo0.decode('ASCII')
if 0:
   i1 * ii1IiI1i % OOooOOo / I11i / o0O / IiiIII111iI
Oo = '476f6f64206a6f622072657665727365722c20707974686f6e206973207665727920636f6f6c21476f6f64206a6f622072657665727365722c20707974686f6e206973207665727920636f6f6c21476f6f64206a6f622072657665727365722c20707974686f6e206973207665727920636f6f6c21'
I1Ii11I1Ii1i = bytes.fromhex(Oo)
Ooo = I1Ii11I1Ii1i.decode('ASCII')
if 0:
   iii1I1I / O00oOoOoO0o0O.O0oo0OO0 + Oo0ooO0oo0oO.I1i1iI1i - II
print o0OO00
# okay decompiling output.pyc
```

![decode1](/assets/img/aster/decode1.png)
![decode2](/assets/img/aster/decode2.png)

so i know the user name searching in msf get us

![module](/assets/img/aster/module.png)
![msfbrute](/assets/img/aster/msfbrute.png)

So we have username and password

```bash
✘ argenestel@parrot  ~/Desktop/tryhackme/aster  telnet 10.10.195.189 5038
Trying 10.10.195.189...
Connected to 10.10.195.189.
Escape character is '^]'.
Asterisk Call Manager/5.0.2
Action: Login
ActionID: 1
Username: admin
Secret: <redacted>


Response: Success
ActionID: 1
Message: Authentication accepted

Event: FullyBooted
Privilege: system,all
Uptime: 1165
LastReload: 1165
Status: Fully Booted
```

```bash
Action: Command
Command: sip show users

Response: Success
Message: Command output follows
Output: Username                   Secret           Accountcode      Def.Context      ACL  Forcerport
Output: 100                        100                               test             No   No        
Output: 101                        101                               test             No   No        
Output: harry                      <redacted>                        test             No   No        
```

```bash
Action: Login
ActionID: 2
Username: harry
Secret: <redacted>
```

![user](/assets/img/aster/aster.png)

#### PrivEsc

So we can see there is Example_root.jar

Using jd-gui..

![javajar](/assets/img/aster/javajar.png)

so we need to get root.txt and /tmp/flag.dat

We will get root.txt
