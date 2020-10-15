---
image: /assets/img/hackpark/hackpark.png
layout: post
title: HackPark
date: 2020-10-6 00:00:01 +5:30
categories: [Tryhackme, Medium]
tags: [] # add tag
---

## Walkthrough

### Enumeration

```bash
argenestel@parrot  ~/Desktop/tryhackme/hackpark  rustscan 10.10.246.43
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
Open 10.10.246.43:80
Open 10.10.246.43:3389
[~] Starting Nmap
[>] The Nmap command to be run is nmap -vvv -p 80,3389 10.10.246.43

Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-12 19:46 IST
Initiating Ping Scan at 19:46
Scanning 10.10.246.43 [2 ports]
Completed Ping Scan at 19:46, 0.27s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 19:46
Completed Parallel DNS resolution of 1 host. at 19:46, 13.00s elapsed
DNS resolution of 1 IPs took 13.00s. Mode: Async [#: 2, OK: 0, NX: 0, DR: 1, SF: 0, TR: 4, CN: 0]
Initiating Connect Scan at 19:46
Scanning 10.10.246.43 [2 ports]
Discovered open port 80/tcp on 10.10.246.43
Discovered open port 3389/tcp on 10.10.246.43
Completed Connect Scan at 19:46, 0.36s elapsed (2 total ports)
Nmap scan report for 10.10.246.43
Host is up, received syn-ack (0.29s latency).
Scanned at 2020-10-12 19:46:31 IST for 14s

PORT     STATE SERVICE       REASON
80/tcp   open  http          syn-ack
3389/tcp open  ms-wbt-server syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 13.79 seconds
```

So the port 80 and 3389 is Open

#### Port 80

![port80](/assets/img/hackpark/port80.png)

So on reverse image search we found the name of clown.
