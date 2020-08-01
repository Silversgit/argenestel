---
layout: post
title: Smag Grotto Writeup
date: 2020-08-02 00:00:00 +5:30
description: The Following Post is writeup of smag grotto room of tryhackme # Add post description (optional)
img: smaggroto.png # Add image post (optional)
tags: [tryhackme, CTF, GTFObins] # add tag
---

## Description:

The Following Post is writeup of smag grotto room of tryhackme <https://tryhackme.com/room/smaggrotto>

## Summary:

  The machine have a webpage which is underdevelopment on dirbusting you will land on a page which contains pcap
  and password upon analysis you will get a subdomain and creds. The admin page have command injection.
  Then the next step is to get user there is cron job running which overwrites the ssh authorized_keys of jack user
  so generate a key and login as jack and then we can run apt-get as root without password.

### Rating: Easy

## Walkthrough:

Lets start with nmap scan

{% highlight ruby %}
nmap -sC -sV <ip>

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 74:e0:e1:b4:05:85:6a:15:68:7e:16:da:f2:c7:6b:ee (RSA)
|   256 bd:43:62:b9:a1:86:51:36:f8:c7:df:f9:0f:63:8f:a3 (ECDSA)
|_  256 f9:e7:da:07:8f:10:af:97:0b:32:87:c9:32:d7:1b:76 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Smag
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
{% endhighlight %}

Start our enum for port 80
we can see a underdevelopment note.

![Port 80](/assets/img/smaggrotto/port80.png)

{% highlight ruby %}
ffuf -w dirb/common.txt -u http://<ip>/FUZZ
index.php               [Status: 200, Size: 402, Words: 69, Lines: 13]
mail                    [Status: 301, Size: 309, Words: 20, Lines: 10]
server-status           [Status: 403, Size: 276, Words: 20, Lines: 10
{% endhighlight %}
http://10.10.247.8/mail/

Here are list of users
netadmin@smag.thm Cc: uzi@smag.thm From: jake@smag.thm.<br/>
We will get a wireshark file. We got 2 info 1st is the subdomain which is development.smag.thm lets add it into /etc/hosts
2nd login creds.

![wireshark analysis](/assets/img/smaggrotto/wireshark.png)<br/>

we can see admin.php have blind command injection which is obviously not that blind<br/>
so i tried reverse shell payload

![Login.php](/assets/img/smaggrotto/login.php.png)<br/>

![Admin Panel](/assets/img/smaggrotto/adminpanel.png)<br/>

![blind command injection](/assets/img/smaggrotto/blindcmd.png)<br/>
so got a blind command injection<br/>
Now start pwncat and let's get a shell

![Shell](/assets/img/smaggrotto/shell.png)<br/>

Priv esc time we can see jack user

It took me a while to see this thing but the line was truly amazing<br/>
*  *    * * *   root    /bin/cat /opt/.backups/jake_id_rsa.pub.backup > /home/jake/.ssh/authorized_keys<br/>
we can write jake backup and hence we will make our id_rsa

so i got user

Matching Defaults entries for jake on smag:<br/>
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jake may run the following commands on smag:<br/>
    (ALL : ALL) NOPASSWD: /usr/bin/apt-get<br/>
Here you go simple gtfo
