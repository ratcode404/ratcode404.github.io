[Back to Main Page](../index.html) 

# Overflow on HackTheBox

![htb](https://www.yeahhub.com/wp-content/uploads/2018/03/hackthebox.png)

## Introduction

It has been quite some time until I have done my last challenge on HTB. Overflow is an older box which requires a cookie analysis and more. Once again, I had booted up Kali, but did not have any strategies just yet.

This write up assumes that the reader is using Kali, but any pentesting distro such as [BlackArch](https://blackarch.org/) will work. The tools come with a stock Kali installation, unless otherwise mentioned.

## 1. Initial Scanning

The first thing we try and as usual is a very basic nmap scan.

```
nmap 10.10.11.119
Nmap scan report for 10.10.11.119
Host is up (0.12s latency).
Not shown: 65532 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
25/tcp open  smtp
80/tcp open  http
```

With further port scans on the ports we receive this information:

```
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 eb:7c:15:8f:f2:cc:d4:26:54:c1:e1:57:0d:d5:b6:7c (RSA)
|   256 d9:5d:22:85:03:de:ad:a0:df:b0:c3:00:aa:87:e8:9c (ECDSA)
|_  256 fa:ec:32:f9:47:17:60:7e:e0:ba:b6:d1:77:fb:07:7b (ED25519)
25/tcp open  smtp    Postfix smtpd
|_smtp-commands: overflow, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, 
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Overflow Sec
Service Info: Host:  overflow; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

SMTP is open, which means we might be able to bruteforce usernames, or send emails. A quick google search on the Apache and SSH versions will tell us that the host is likely running Ubuntu 18.04 Bionic.



