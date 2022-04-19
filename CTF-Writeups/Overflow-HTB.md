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

Checking the webpage that is hosted on the IP, we find this.

![webpage](https://i.imgur.com/SZNrJug.png)

There's not much there other than 'Sign In' and 'Sign Up' links, and a 'Contact Us' form at the bottom. Submitting the 'Contact Us' form just sends a GET to /?, so it doesn't do anything important. But the login redirects to `/login.php` and signup to `/register.php`.

![login](https://i.imgur.com/AkuQ67a.png)

Playing a round with both masks, there was no quick solution to get through these with simple injections. Unfortunately, there was no way to enumerate the usernames through checking the error messages in the login mask, but there is a different approach. Trying to register as admin will return to the registration.php, whereas any successful registration gets a 302: This means we are able to enumerate the users this way.

Next, we register an account, which redirects back to `/home/index.php`, the front page. It is very similar to the page before, but the menu offers various options instead. 

![menu](https://i.imgur.com/dVdkApn.png)

There is also a profile page at `/home/profile/`, but the buttons there do visibly not work.

![profile](https://i.imgur.com/ZVzBYzM.png)

`/home/blog.php` has four different posts, but the links do not work once again.

![blog](https://i.imgur.com/2gIcpky.png)

Other than that, there is also the `/logout.php`, which deletes the auth cookie and returns a 302 to the homepage.

```
HTTP/1.1 302 Found
Date: Mon, 27 Sep 2021 20:27:05 GMT
Server: Apache/2.4.29 (Ubuntu)
Set-Cookie: auth=deleted; expires=Thu, 01-Jan-1970 00:00:01 GMT; Max-Age=0
Location: index.php
Content-Length: 0
Connection: close
Content-Type: text/html; charset=UTF-8
```
