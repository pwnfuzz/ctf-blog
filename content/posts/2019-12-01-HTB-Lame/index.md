---
"title": "Hack The Box - Lame"
"date": 2019-12-01
"tags": ["linux", "easy", "SUID", "cve"]
"keywords": ["linux", "easy", "SUID", "cve"]
"categories": ["HackTheBox OSCP-Like"]
"author": "Ghostbyt3"
"description": "We are going to pwn Lame from Hack The Box."
"featured_image": "/img/htb-lame/1.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox OSCP-Like - Writeup"
---


![Untitled](/img/htb-lame/1.png)

We are going to pwn Lame from Hack The Box.

Link : [https://www.hackthebox.eu/home/machines/profile/1](https://www.hackthebox.eu/home/machines/profile/1)


Lets Begin with our Initial Nmap Scan.

## Nmap Scan Results
```bash
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3632/tcp open  distccd

PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.7
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
3632/tcp open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: remote management|WAP|printer|general purpose|power-device
Running (JUST GUESSING): Dell embedded (92%), Linksys embedded (92%), Tranzeo embedded (92%), Xerox embedded (92%), Linux 2.4.X|2.6.X (92%), Dell iDRAC 6 (92%), Raritan embedded (92%)
OS CPE: cpe:/h:dell:remote_access_card:6 cpe:/h:linksys:wet54gs5 cpe:/h:tranzeo:tr-cpq-19f cpe:/h:xerox:workcentre_pro_265 cpe:/o:linux:linux_kernel:2.4 cpe:/o:linux:linux_kernel:2.6 cpe:/o:dell:idrac6_firmware cpe:/o:linux:linux_kernel:2.6.22
Aggressive OS guesses: Dell Integrated Remote Access Controller (iDRAC6) (92%), Linksys WET54GS5 WAP, Tranzeo TR-CPQ-19f WAP, or Xerox WorkCentre Pro 265 printer (92%), Linux 2.4.21 - 2.4.31 (likely embedded) (92%), Linux 2.6.8 - 2.6.30 (92%), Dell iDRAC 6 remote access controller (Linux 2.6) (92%), Raritan Dominion PX DPXR20-20L power control unit (92%), LifeSize video conferencing system (Linux 2.4.21) (92%), Linux 2.6.23 (91%), OpenWrt Kamikaze 7.09 (Linux 2.6.22) (90%), Arris TG862G/CT cable modem (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

## FTP Enumeration

So lets start from 1st port ```ftp``` 
Lets check whether the version has any exploit available.

Since we know the verion of ``ftp`` which is ``vsftpd 2.3.4`` from nmap scan lets search for any exploits available.
![Untitled](/img/htb-lame/2.png)

It didn't worked, they might patched it.
![Untitled](/img/htb-lame/3.png)

Lets check other ports.

## SMB Enumeration

First I checked any shares available and it seems we have Read and Write access on `tmp`.
```bash
root@kali:~# smbmap -H 10.10.10.3
[+] IP: 10.10.10.3:445	Name: 10.10.10.3                                        
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	print$                                            	NO ACCESS	Printer Drivers
	tmp                                               	READ, WRITE	oh noes!
	opt                                               	NO ACCESS	
	IPC$                                              	NO ACCESS	IPC Service (lame server (Samba 3.0.20-Debian))
	ADMIN$                                            	NO ACCESS	IPC Service (lame server (Samba 3.0.20-Debian))
```
I get some error if I tried connecting to it.

## Getting User Shell

We know port `3632` running `Distcc` So I googled for any exploits available and found an exploit/

> https://www.rapid7.com/db/modules/exploit/unix/misc/distcc_exec

>use exploit/unix/misc/distcc_exec

![Untitled](/img/htb-lame/4.png)

We got shell and found an user ``makis``[br/](br/)
![Untitled](/img/htb-lame/5.png)

### Without Metasploit

There is a python script from Github
> https://gist.github.com/DarkCoderSc/4dbf6229a93e75c3bdf6b467e67a9855

I tried using that python script and didn't changed any and got the shell as `daemon`.
![Untitled](/img/htb-lame/8.png)

## Privilege Escalation

Uploaded my enumeration Script and 
Found some SETUID.[br/](br/)
![Untitled](/img/htb-lame/6.png)

Lets try with ``nmap``

I searched for nmap in GTFOBins 

>https://gtfobins.github.io/gtfobins/nmap/

```
nmap --interactive
nmap> !sh
```
Using the commands

![Untitled](/img/htb-lame/7.png)

We got the Root!!

