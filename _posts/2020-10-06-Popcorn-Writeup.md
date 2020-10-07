---
image: /assets/img/popcorn/popcorn.png
layout: post
title: Popcorn
date: 2020-10-6 00:00:01 +5:30
categories: [Hackthebox, Medium]
tags: [torrent, web, upload] # add tag
---

## Summary

> We have torrent in port 80
> Created an Account
> There is an upload vulnerability in screenshot upload feature
> After getting shell, linuxexpliotsuggester will show some exploits
> Exploiting with nelson Exploit will give Root

## Walkthrough

### Enumeration

```bash
argenestel@parrot  ~/Desktop/hackthebox/popcorn  rustscan 10.10.10.6
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
Faster Nmap scanning with Rust.
________________________________________
: https://discord.gg/GFrQsGy           :
: https://github.com/RustScan/RustScan :
 --------------------------------------
Nmap? More like slowmap.

[~] The config file is expected to be at "/home/argenestel/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5
000'.  
Open 10.10.10.6:22
Open 10.10.10.6:80
[~] Starting Nmap
[>] The Nmap command to be run is nmap -vvv -p 22,80 10.10.10.6

Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-04 21:09 IST
Initiating Ping Scan at 21:09
Scanning 10.10.10.6 [2 ports]
Completed Ping Scan at 21:09, 0.21s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 21:09
Completed Parallel DNS resolution of 1 host. at 21:09, 0.05s elapsed
DNS resolution of 1 IPs took 0.05s. Mode: Async [#: 2, OK: 0, NX: 1, DR: 0, SF: 0, TR: 1, CN: 0]
Initiating Connect Scan at 21:09
Scanning 10.10.10.6 [2 ports]
Discovered open port 80/tcp on 10.10.10.6
Discovered open port 22/tcp on 10.10.10.6
Completed Connect Scan at 21:09, 0.20s elapsed (2 total ports)
Nmap scan report for 10.10.10.6
Host is up, received syn-ack (0.20s latency).
Scanned at 2020-10-04 21:09:37 IST for 1s

PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack
80/tcp open  http    syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.57 seconds

```

#### Port 80

Okay  let's dirbust the port80.

```bash
argenestel@parrot  ~/Desktop/hackthebox/popcorn  ffuf -w /usr/share/wordlists/dirb/big.txt -u http://10.10.10.6/FUZZ  

       /'___\  /'___\           /'___\        
      /\ \__/ /\ \__/  __  __  /\ \__/        
      \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\       
       \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/       
        \ \_\   \ \_\  \ \____/  \ \_\        
         \/_/    \/_/   \/___/    \/_/        

      v1.0.2
________________________________________________

:: Method           : GET
:: URL              : http://10.10.10.6/FUZZ
:: Follow redirects : false
:: Calibration      : false
:: Timeout          : 10
:: Threads          : 40
:: Matcher          : Response status: 200,204,301,302,307,401,403
________________________________________________

.htaccess               [Status: 403, Size: 287, Words: 21, Lines: 11]
.htpasswd               [Status: 403, Size: 287, Words: 21, Lines: 11]
cgi-bin/                [Status: 403, Size: 286, Words: 21, Lines: 11]
index                   [Status: 200, Size: 177, Words: 22, Lines: 5]
rename                  [Status: 301, Size: 309, Words: 20, Lines: 10]
test                    [Status: 200, Size: 47067, Words: 2465, Lines: 651]
torrent                 [Status: 301, Size: 310, Words: 20, Lines: 10]
```

![port](/assets/img/popcorn/port80torrent.png)

![login](/assets/img/popcorn/login.png)

Alright we checked common passwords and didn't worked.<br/>
Let's check by reg user (admin user already exists).

![reg](/assets/img/popcorn/registered.png)

Okay let's check if uploading php is possible or not<br />
ughh we can only upload torrent file.
So looking into previously uploaded file we can upload screenshot to let's check by uploading a torrent file.

since it is checking content type i changed it from application/x-php to image/png
and it was accepted as png

![upload](/assets/img/popcorn/uploaded.png)

Alright done now lets manupulate screenshot.

![uploadpng](/assets/img/popcorn/uploadedpng.png)

![png](/assets/img/popcorn/pngerror.png)

### Exploitation

Hurrayy let's get shell

```bash
argenestel@parrot  ~/Desktop/hackthebox/popcorn  nc -lvnp  4444  
listening on [any] 4444 ...
connect to [10.10.14.10] from (UNKNOWN) [10.10.10.6] 48833
Linux popcorn 2.6.31-14-generic-pae #48-Ubuntu SMP Fri Oct 16 15:22:42 UTC 2009 i686 GNU/Linux
 21:47:21 up  4:03,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
bash: no job control in this shell
www-data@popcorn:/$ python -c 'import pty;pty.spawn("/bin/bash")'
python -c 'import pty;pty.spawn("/bin/bash")'
www-data@popcorn:/$ id
id
```

alright got www-data<br />
moving on to user

so we got george username<br />

![reg](/assets/img/popcorn/linuxexploitsuggester.png)

![reg](/assets/img/popcorn/nelsonexp.png)

<https://github.com/lucyoa/kernel-exploits/blob/master/full-nelson/full-nelson> <br />
so i got root by running linux exploit suggester.

Rooted :)
