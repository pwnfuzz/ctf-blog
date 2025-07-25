---
"title": "Hack The Box - Irked"
"date": 2019-12-06
"tags": ["linux", "easy", "steg"]
"keywords": ["linux", "easy", "steg"]
"categories": ["HackTheBox"]
"author": "Ghostbyt3"
"description": "We are going to pwn Irked from Hack The Box."
"featured_image": "/img/htb-irked/1.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox - Writeup"
---


![Untitled](/img/htb-irked/1.png)

We are going to pwn Irked from Hack The Box.

Link : [https://www.hackthebox.eu/home/machines/profile/163](https://www.hackthebox.eu/home/machines/profile/163)


Like always begin with our Nmap Scan.

## Nmap Scan Results

```bash
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
111/tcp   open  rpcbind
6697/tcp  open  ircs-u
8067/tcp  open  infi-async
60236/tcp open  unknown
65534/tcp open  unknown


PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey: 
|   1024 6a:5d:f5:bd:cf:83:78:b6:75:31:9b:dc:79:c5:fd:ad (DSA)
|   2048 75:2e:66:bf:b9:3c:cc:f7:7e:84:8a:8b:f0:81:02:33 (RSA)
|   256 c8:a3:a2:5e:34:9a:c4:9b:90:53:f7:50:bf:ea:25:3b (ECDSA)
|_  256 8d:1b:43:c7:d0:1a:4c:05:cf:82:ed:c1:01:63:a2:0c (ED25519)
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Site doesn't have a title (text/html).
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          39016/udp6  status
|   100024  1          45688/udp   status
|   100024  1          52691/tcp6  status
|_  100024  1          60236/tcp   status
6697/tcp  open  irc     UnrealIRCd
8067/tcp  open  irc     UnrealIRCd
60236/tcp open  status  1 (RPC #100024)
65534/tcp open  irc     UnrealIRCd
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.12 (95%), Linux 3.13 (95%), Linux 3.2 - 4.9 (95%), Linux 3.8 - 3.11 (95%), Linux 4.8 (95%), Linux 4.4 (95%), Linux 3.16 (95%), Linux 3.18 (95%), Linux 4.2 (95%), ASUS RT-N56U WAP (Linux 3.4) (95%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: irked.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Start from HTTP port 
![Untitled](/img/htb-irked/2.png)[br/](br/)
This image looks suspecious , lets try Gobuster

## Gobuster Results

```bash
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.117
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2019/12/10 23:56:59 Starting gobuster
===============================================================
/.hta (Status: 403)
/.htaccess (Status: 403)
/.htpasswd (Status: 403)
/index.html (Status: 200)
/manual (Status: 301)
/server-status (Status: 403)
===============================================================
2019/12/10 23:59:11 Finished
===============================================================
```

Nothing useful. Check other ports.
We know ``UnrealIRCD`` running, search them in searchsploit for any exploits
![Untitled](/img/htb-irked/3.png)

I choose randomly and started with ``metasploit`` one

>https://www.rapid7.com/db/modules/exploit/unix/irc/unreal_ircd_3281_backdoor

>use exploit/unix/irc/unreal_ircd_3281_backdoor

## Getting Shell

![Untitled](/img/htb-irked/4.png)[br/](br/)
Set RHOST and LHOST

I got the shell as ``ircd``[br/](br/)
![Untitled](/img/htb-irked/5.png)[br/](br/)
While checking the home directory I found an user ``djmardov`` and got ``.backup`` file. 
It says ``steg`` maybe it is ``Stegnanography``

## Getting User Djmardov

We already saw an image in webpage , download it
I tried with [steghide](https://github.com/StefanoDeVuono/steghide)

>steghide - a steganography program

![Untitled](/img/htb-irked/6.png)

It has a file called ``pass.txt``which is password protected and we got the pass already from ``.backup``

I got a password

![Untitled](/img/htb-irked/7.png)

```
extract, --extract      extract data
-xf, --extractfile      select file name for extracted data
```

So I tried with them in ``ssh`` - ``djmardov:Kab6h+m+bbp2J:HG``[br/](br/)
![Untitled](/img/htb-irked/8.png)


## Privilege Escalation

There is an setuid binary ``viewuser``[br/](br/)
![Untitled](/img/htb-irked/9.png)

While executing it search for a file called ``listusers`` in ``/tmp``[br/](br/)
![Untitled](/img/htb-irked/10.png)

Which is not found so we create one and make that gives us shell.[br/](br/)
![Untitled](/img/htb-irked/11.png)

I created one and tried executed the file ``viewuser`` it says permission denied for ``listusers``, I forgot to give permission and then it worked.
> echo “/bin/sh” > listusers

I got Root~



