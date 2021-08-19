[Back to Main Page](../index.html) 

# Ophiuchi on HackTheBox


![htb](https://www.yeahhub.com/wp-content/uploads/2018/03/hackthebox.png)



## Introduction

Bashed is a medium box which was a CTF I attempted after a rather long break.

This write up assumes that the reader is using Kali, but any pentesting distro such as [BlackArch](https://blackarch.org/) will work. The tools come with a stock Kali installation, unless otherwise mentioned.

## 1. Initial Scanning

All we do have is an IP so we start to run a nmapAutomator scan.

`nmap 10.10.10.68`

nmap did find two ports.
* 22/tcp open ssh OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
* 8080/tcp open http Apache Tomcat 9.0.38

Browsing to the website hosted on the Apache Tomcat, I found out that it is an online parser for YAML. 

![1ophiuchi](https://i.imgur.com/cGWoc4Z.png)

## 2. SnakeYAML deserialization exploit

After a bit of abusing google to figure out what YAML exactly is, and how to exploit it, I stumbled over the 'SnakeYAML deserialization exploit'.

![2ophiuchi](https://i.imgur.com/Gu0RJ3v.png)

We can use this deserialization vulnerablity to get remote code execution. 

WIP


## 5. Cleanup
 
Since this is a CTF, cleanup isn’t mandatory.
However, we want to develop good habits and operational security practice. Meterpreter has a `clearev` command that can be used to cover our tracks - let’s run it and be out of here.

![hacked](https://cdn-images-1.medium.com/max/1600/1*TtuBByJ52bSP-d2WPOczJg.png)
