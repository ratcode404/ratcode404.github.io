[Back to Main Page](../index.html) 

# Overflow on HackTheBox

![htb](https://www.yeahhub.com/wp-content/uploads/2018/03/hackthebox.png)

## Introduction

It has been quite some time until I have done my last challenge on HTB. Overflow is an older box which requires a cookie analysis and more. Once again, I had booted up Kali, but did not have any strategies just yet.

This write up assumes that the reader is using Kali, but any pentesting distro such as [BlackArch](https://blackarch.org/) will work. The tools come with a stock Kali installation, unless otherwise mentioned.

## 1. Initial Scanning

The first thing we try and as usual is a very basic nmap scan.

`
nmap 10.10.11.119
Nmap scan report for 10.10.11.119
Host is up (0.12s latency).
Not shown: 65532 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
25/tcp open  smtp
80/tcp open  http
`

sadsa
