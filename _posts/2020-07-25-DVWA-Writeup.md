---
layout: post
title: DVWA
date: 2020-07-25 00:00:00 +5:30
categories: [WebApp]
tags: [Web] # add tag
---

## Description

So i am using TryHackMe room to Complete DVWA which is in my list since i started but never Complete it.

## Low Security:

Let's start with low security level

### Bruteforce

So we have a login page and as we know it isn't secured we can Bruteforce.<br />
I am using burp intruder and fasttrack.txt.

and here we got Password

### Command Injection

The web have a prompt which say we can ping an IP.
So we can say maybe the background code is "ping <input>" and we are running in linux.

Okay got the injection well.

### File Inclusion

A simple LFI by changing the ?page= we can get any while in system without restrictions.

To be continued....
