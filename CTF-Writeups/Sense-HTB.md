[Back to Main Page](../index.html) 

# Bashed on HackTheBox


![htb](https://www.yeahhub.com/wp-content/uploads/2018/03/hackthebox.png)



## Introduction

Sense is a beginner's box which was the third CTF I ever attempted. I still had near to no experience when trying to capture the flag.

This write up assumes that the reader is using Kali, but any pentesting distro such as [BlackArch](https://blackarch.org/) will work. The tools come with a stock Kali installation, unless otherwise mentioned.

## 1. Initial Scanning


First I started as usual an nmap scan `nmap 10.10.10.60`.

![sense1](https://i.imgur.com/tFfFfdC.png)

Port 80 and 443 tcp are open, so I started enumerating the whole server with dirbuster.   
Wordlist: directory-list-lowercase-2.3-medium.txt   
File extension: php, html,txt,sh,xml

![sense2](https://i.imgur.com/88ell4K.png)

After over an hour I found something interesting.

![sense3](https://i.imgur.com/7nmJRUG.png)

`/installer/system-users.txt`  

![sense4](https://i.imgur.com/J4CNI3S.png)

Wow, a user in a .txt. Not the best way to store your data. Password is pfsense, as it's the standard password for opnsense.

After login, I did not find anything usable inside the GUI, so I decided to search for exploits. Lucky @spencerdodd made an easy version of an fitting exploit. There are many other versions of this exploit too, but this seemed the best for me.

https://github.com/spencerdodd/pfsense-code-exec

`pfSense Community Edition firewall version 2.2.6 and below is vulnerable to arbitrary code execution exploit as an authenticated non-administrative user. The initial advisory came from Security Assessment in April 2016, however until very recently there was not a public exploit for this vulnerability. This is my version of this exploit.`

To use the exploit, I edited the .py file, to match it my settings, just like this.

![sense5](https://i.imgur.com/0stBrcz.png)

After this, I created an listener for port 4444, and used the .py script. I used the -nc Parameter because I did not want to use msf, and had the nc listener already up. So I can't stay if -msf works too.

`nc -lvp 4444`    
`python3 pfsense_exec.py -nc`

![sense6](https://i.imgur.com/zC12Al7.png)

And we have root access, now just capture the flag and we're done!

![sense7](https://i.imgur.com/b0S4mEm.png)



## 5. Cleanup
 
Since this is a CTF, cleanup isn’t mandatory.
However, we want to develop good habits and operational security practice. Meterpreter has a `clearev` command that can be used to cover our tracks - let’s run it and be out of here.

![hacked](https://cdn-images-1.medium.com/max/1600/1*TtuBByJ52bSP-d2WPOczJg.png)
