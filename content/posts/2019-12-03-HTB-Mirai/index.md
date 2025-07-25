---
"title": "Hack The Box - Mirai"
"date": 2019-12-03
"tags": ["linux", "easy", "sudo"]
"keywords": ["linux", "easy", "sudo"]
"categories": "HackTheBox"
"author": "Ghostbyt3"
"description": "We are going to pwn Mirai from Hack The Box."
"featured_image": "/img/htb-mirai/1.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox - Writeup"
---


![Untitled](/img/htb-mirai/1.png)

We are going to pwn Mirai from Hack The Box.

Link : [https://www.hackthebox.eu/home/machines/profile/64](https://www.hackthebox.eu/home/machines/profile/64)


Like always begin with our Nmap Scan.

## Nmap Scan Results
```bash
PORT      STATE SERVICE
22/tcp    open  ssh
53/tcp    open  domain
80/tcp    open  http
1497/tcp  open  rfx-lm
32400/tcp open  plex
32469/tcp open  unknown


PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u3 (protocol 2.0)
| ssh-hostkey: 
|   1024 aa:ef:5c:e0:8e:86:97:82:47:ff:4a:e5:40:18:90:c5 (DSA)
|   2048 e8:c1:9d:c5:43:ab:fe:61:23:3b:d7:e4:af:9b:74:18 (RSA)
|   256 b6:a0:78:38:d0:c8:10:94:8b:44:b2:ea:a0:17:42:2b (ECDSA)
|_  256 4d:68:40:f7:20:c4:e5:52:80:7a:44:38:b8:a2:a7:52 (ED25519)
53/tcp    open  domain  dnsmasq 2.76
| dns-nsid: 
|_  bind.version: dnsmasq-2.76
80/tcp    open  http    lighttpd 1.4.35
|_http-server-header: lighttpd/1.4.35
|_http-title: Website Blocked
1497/tcp  open  upnp    Platinum UPnP 1.0.5.13 (UPnP/1.0 DLNADOC/1.50)
32400/tcp open  http    Plex Media Server httpd
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Server returned status 401 but no WWW-Authenticate header.
|_http-cors: HEAD GET POST PUT DELETE OPTIONS
|_http-title: Unauthorized
32469/tcp open  upnp    Platinum UPnP 1.0.5.13 (UPnP/1.0 DLNADOC/1.50)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.12 (95%), Linux 3.13 (95%), Linux 3.16 (95%), Linux 3.18 (95%), Linux 3.2 - 4.9 (95%), Linux 3.8 - 3.11 (95%), Linux 4.8 (95%), Linux 4.4 (95%), Linux 4.2 (95%), ASUS RT-N56U WAP (Linux 3.4) (95%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Lets begin with HTTP port, it looks like empty so we can try GoBuster
![Untitled](/img/htb-mirai/2.png)

## Gobuster Results
```bash
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.48
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2019/12/04 18:21:53 Starting gobuster
===============================================================
/admin (Status: 301)
/swfobject.js (Status: 200)
===============================================================
2019/12/04 18:24:03 Finished
===============================================================
```
Looks like we’re at a page for Pi-hole
![Untitled](/img/htb-mirai/3.png)

So I googled for any default credentials.[br/](br/)
![Untitled](/img/htb-mirai/4.png)

Clicking on the Login button from the Pi-hole page, when I enter ``raspberry`` as the password I get a login failed.

![Untitled](/img/htb-mirai/5.png)

## Getting Shell
So I tried with ``ssh`` and it worked!!
![Untitled](/img/htb-mirai/6.png)

## Privilege Escaltion

Like always I start with ``sudo -l`` and it looks like i can run sudo command without password
![Untitled](/img/htb-mirai/7.png)

Im root now!!

![Untitled](/img/htb-mirai/8.png)

But while seeing the ``root flag`` it gives some message.
It looks like ``flag`` is in ``USB Stick``
![Untitled](/img/htb-mirai/9.png)

## Finding the missing Flag

So, now we need to look for a USB drive location and it is located in ``/media/usbstick``
But it is also deleted from there

![Untitled](/img/htb-mirai/10.png)

We can do ``df -lh`` to check the space available on a particular file system.

> df - report file system disk space usage

![Untitled](/img/htb-mirai/11.png)

So we can do ``strings`` Just to check anything available on file system and I came to know for ``/media/usbstick`` ``/dev/sdb`` is the file system.

![Untitled](/img/htb-mirai/12.png)

We got the Root Flag!!


