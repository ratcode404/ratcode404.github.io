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

We can use this deserialization vulnerablity to get remote code execution - For this I used and cloned [yaml payload on GitHub](https://github.com/artsploit/yaml-payload).

`git clone https://github.com/artsploit/yaml-payload`

![3ophiuchi](https://i.imgur.com/RhMPiJH.png | width=650)

There we modified the AwesomeScriptEngineFactory() method within AwesomeScriptEngineFactory.java. We can use the payloads found [Issue-3](https://github.com/artsploit/yaml-payload/issues/3) with our IP. Now as explained in the repository, I compiled the AwesomeScriptEngineFactory.java and build the yaml-payload.jar file.

![4ophiuchi](https://i.imgur.com/zG8TUn8.png)

This payload I copied to the apache directory and made sure the server is running.

`cp yaml-payload.jar /var/www/html/yaml-payload.jar && sudo service apache2 start`

Now, we should be able to execute the YAML parser on the victim's machine, after starting the netcast listener on port 8081.

![5ophiuchi](https://imgur.com/a/hmjnLRd)

One I clicked the PARSE button, the payload was executed and I had access to the reverse shell as the tomcat user.

![6ophiuchi](https://i.imgur.com/tJ79z7A.png)

## 3. Privilege Escalation - User

Finding the user flag is fairly simple. After a basic enumeration we'll find that /opt/tomcat/conf/tomcat-users.xml contains a password for the admin user.

`<user username="admin" password="whythereisalimit" roles="manager-gui,admin-gui"/>`

We can now ssh into the machine as suer admin using the obtained creds.

## 4. Privilege Escalation - root

First, I checked what sudo capabilities our user admin got.

`sudo -l
(ALL) NOPASSWD: /usr/bin/go run /opt/wasm-functions/index.go`

So we can run /usr/bin/go run /opt/wasm-functions/index.go with root privileges. Let’s check out the file.



## 5. Cleanup
 
Since this is a CTF, cleanup isn’t mandatory.
However, we want to develop good habits and operational security practice. Meterpreter has a `clearev` command that can be used to cover our tracks - let’s run it and be out of here.

![hacked](https://cdn-images-1.medium.com/max/1600/1*TtuBByJ52bSP-d2WPOczJg.png)
