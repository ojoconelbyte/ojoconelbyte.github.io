---
date: 2024-04-16 22:54:50
layout: post
title: "HTB WriteUp - Networked"
subtitle: Guia para resolver esta maquina Linux - Easy de HackTheBox
description: Maquina Linux Easy de HTB
image: /assets/img/uploads/networked.png
optimized_image:
category: pen-testing
tags: HTB Easy
author: ojoconelbyte
paginate: false
---
# Networked 

### Machine Info & Overview!
- Categories -> Vulnerability Assessment Web Application
- Area of Interest -> Source Code Analysis Injections
- Vulnerabilities -> OS Command Injection Arbitrary File Upload
- Languages -> PHP 

# Resolution summary

- Networked is an Easy difficulty Linux box vulnerable to file upload bypass, leading to code execution. Due to improper sanitization, a crontab running as the user can be exploited to achieve command execution. The user has privileges to execute a network configuration script, which can be leveraged to execute commands as root. 
## Improved skills
- skill 1
- skill 2

## Used tools
- nmap
- gobuster

---

# Information Gathering

Scanned all TCP ports:
```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.10.146 -oG allPorts

Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-04-16 19:43 WEST
Initiating SYN Stealth Scan at 19:43
Scanning 10.10.10.146 [65535 ports]
Discovered open port 22/tcp on 10.10.10.146
Discovered open port 80/tcp on 10.10.10.146
Completed SYN Stealth Scan at 19:44, 26.49s elapsed (65535 total ports)
Nmap scan report for 10.10.10.146
Host is up, received user-set (0.23s latency).
Scanned at 2024-04-16 19:43:50 WEST for 26s
Not shown: 65500 filtered tcp ports (no-response), 32 filtered tcp ports (host-prohibited), 1 closed tcp port (reset)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 26.56 seconds
           Raw packets sent: 131043 (5.766MB) | Rcvd: 35 (2.432KB)
```

Enumerated open TCP ports:
```bash
nmap -sCV -p22,80 10.10.10.146 -oG allPorts

Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-04-16 19:45 WEST
Nmap scan report for 10.10.10.146
Host is up (0.22s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 22:75:d7:a7:4f:81:a7:af:52:66:e5:27:44:b1:01:5b (RSA)
|   256 2d:63:28:fc:a2:99:c7:d4:35:b9:45:9a:4b:38:f9:c8 (ECDSA)
|_  256 73:cd:a0:5b:84:10:7d:a7:1c:7c:61:1d:f5:54:cf:c4 (ED25519)
80/tcp open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.80 seconds
```

Enumerating directories used by popular web applications and servers:
```bash
nmap --script http-enum -p80 10.10.10.146 -oN webScan 

Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-04-16 20:31 WEST
Nmap scan report for 10.10.10.146
Host is up (0.22s latency).

PORT   STATE SERVICE
80/tcp open  http
| http-enum: 
|   /backup/: Backup folder w/ directory listing
|   /icons/: Potentially interesting folder w/ directory listing
|_  /uploads/: Potentially interesting folder

Nmap done: 1 IP address (1 host up) scanned in 25.50 seconds
```


---

# Enumeration
## Port 80 - HTTP (Apache 2.4.6)
Lorem ipsum dolor sit amet, consectetur adipiscing elit. Quisque sit amet tortor scelerisque, fringilla sapien sit amet, rhoncus lorem. Nullam imperdiet nisi ut tortor eleifend tincidunt. Mauris in aliquam orci. Nam congue sollicitudin ex, sit amet placerat ipsum congue quis. Maecenas et ligula et libero congue sollicitudin non eget neque. Phasellus bibendum ornare magna. Donec a gravida lacus.

---

# Exploitation
## Name of the technique
Lorem ipsum dolor sit amet, consectetur adipiscing elit. Quisque sit amet tortor scelerisque, fringilla sapien sit amet, rhoncus lorem. Nullam imperdiet nisi ut tortor eleifend tincidunt. Mauris in aliquam orci. Nam congue sollicitudin ex, sit amet placerat ipsum congue quis. Maecenas et ligula et libero congue sollicitudin non eget neque. Phasellus bibendum ornare magna. Donec a gravida lacus.

---

# Lateral Movement to user
## Local Enumeration
Lorem ipsum dolor sit amet, consectetur adipiscing elit. Quisque sit amet tortor scelerisque, fringilla sapien sit amet, rhoncus lorem. Nullam imperdiet nisi ut tortor eleifend tincidunt. Mauris in aliquam orci. Nam congue sollicitudin ex, sit amet placerat ipsum congue quis. Maecenas et ligula et libero congue sollicitudin non eget neque. Phasellus bibendum ornare magna. Donec a gravida lacus.

## Lateral Movement vector
Lorem ipsum dolor sit amet, consectetur adipiscing elit. Quisque sit amet tortor scelerisque, fringilla sapien sit amet, rhoncus lorem. Nullam imperdiet nisi ut tortor eleifend tincidunt. Mauris in aliquam orci. Nam congue sollicitudin ex, sit amet placerat ipsum congue quis. Maecenas et ligula et libero congue sollicitudin non eget neque. Phasellus bibendum ornare magna. Donec a gravida lacus.

---

# Privilege Escalation
## Local Enumeration
Lorem ipsum dolor sit amet, consectetur adipiscing elit. Quisque sit amet tortor scelerisque, fringilla sapien sit amet, rhoncus lorem. Nullam imperdiet nisi ut tortor eleifend tincidunt. Mauris in aliquam orci. Nam congue sollicitudin ex, sit amet placerat ipsum congue quis. Maecenas et ligula et libero congue sollicitudin non eget neque. Phasellus bibendum ornare magna. Donec a gravida lacus.

## Privilege Escalation vector
Lorem ipsum dolor sit amet, consectetur adipiscing elit. Quisque sit amet tortor scelerisque, fringilla sapien sit amet, rhoncus lorem. Nullam imperdiet nisi ut tortor eleifend tincidunt. Mauris in aliquam orci. Nam congue sollicitudin ex, sit amet placerat ipsum congue quis. Maecenas et ligula et libero congue sollicitudin non eget neque. Phasellus bibendum ornare magna. Donec a gravida lacus.

---

# Trophy & Loot
user.txt

root.txt
