---
"title": "Hack The Box - Bank"
"date": 2019-12-02
"tags": ["linux", "easy", "setuid", "redirection"]
"keywords": ["linux", "easy", "setuid", "redirection"]
"categories": ["HackTheBox"]
"author": "Ghostbyt3"
"description": "We are going to pwn Bank from Hack The Box."
"featured_image": "/img/htb-bank/1.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox - Writeup"
---


![Untitled](/img/htb-bank/1.png)

We are going to pwn Bank from Hack The Box.

Link : [https://www.hackthebox.eu/home/machines/profile/26](https://www.hackthebox.eu/home/machines/profile/26)


Like always begin with our Nmap Scan.

## Nmap Scan Results
```bash
PORT   STATE SERVICE
22/tcp open  ssh
53/tcp open  domain
80/tcp open  http

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 08:ee:d0:30:d5:45:e4:59:db:4d:54:a8:dc:5c:ef:15 (DSA)
|   2048 b8:e0:15:48:2d:0d:f0:f1:73:33:b7:81:64:08:4a:91 (RSA)
|   256 a0:4c:94:d1:7b:6e:a8:fd:07:fe:11:eb:88:d5:16:65 (ECDSA)
|_  256 2d:79:44:30:c8:bb:5e:8f:07:cf:5b:72:ef:a1:6d:67 (ED25519)
53/tcp open  domain  ISC BIND 9.9.5-3ubuntu0.14 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.9.5-3ubuntu0.14-Ubuntu
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.12 (95%), Linux 3.13 (95%), Linux 3.16 (95%), Linux 3.18 (95%), Linux 3.2 - 4.9 (95%), Linux 3.8 - 3.11 (95%), Linux 4.4 (95%), Linux 4.2 (95%), Linux 4.8 (95%), ASUS RT-N56U WAP (Linux 3.4) (95%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

When I see the Webpage its default apache2 server so I added ``10.10.10.29 bank.htb`` in ``/etc/hosts`` then when I open ``bank.htb`` I got an login page.
![Untitled](/img/htb-bank/2.png)

We have a webpage so why not running ``Gobuster``

## Gobuster Results
```bash
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://bank.htb
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/01/26 00:04:15 Starting gobuster
===============================================================
/.hta (Status: 403)
/.htaccess (Status: 403)
/.htpasswd (Status: 403)
/assets (Status: 301)
/inc (Status: 301)
/index.php (Status: 302)
/server-status (Status: 403)
/uploads (Status: 301)
/support.php (Status: 302)
===============================================================
2020/01/26 00:06:33 Finished
===============================================================
```
From the scan results i tried ``/index.php`` but it redirects to ``/login.php`` so lets switch on the burp and see whats going on!
![Untitled](/img/htb-bank/3.png)

Yes when we try to open ``/index.php`` a ``302 Found`` which is redirection, We can stop a redirection using burp so lets do that.

All we need to do is change ``302 Found`` to ``200 Ok``
For that open `` Proxy -> Options -> Match and Replace ``

![Untitled](/img/htb-bank/4.png)
![Untitled](/img/htb-bank/5.png)

Add Now Redirection is stopped , Let see whats in the webpage[br/](br/)
![Untitled](/img/htb-bank/6.png)
It looks like some users bank account details

## Finding Upload Vulnerability

There is another webpage ``support.php`` Lets see whats inside.[br/](br/)
![Untitled](/img/htb-bank/7.png)

It looks like some kind of upload thing. While Checking the source code found this[br/](br/)
![Untitled](/img/htb-bank/8.png)

## Getting Shell

So I uploaded my payload ``.php`` as ``.htb`` [br/](br/)
![Untitled](/img/htb-bank/13.png)

While Listening on my machine and Pressing the Click Here opened the shell for me!
![Untitled](/img/htb-bank/9.png)

While enumerating I found some credentials which is located in ``/www/bank``[br/](br/)
![Untitled](/img/htb-bank/10.png)

## Privilege Escalation

Found an SUID called ``emergency`` in ``/var/htb/bin``[br/](br/)
![Untitled](/img/htb-bank/11.png)

While I tried to execute it give me ``root`` 
![Untitled](/img/htb-bank/12.png)






