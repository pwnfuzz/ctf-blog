---
"title": "Hack The Box - Networked"
"date": 2019-12-14
"tags": ["linux", "easy", "php", "sudo"]
"keywords": ["linux", "easy", "php", "sudo"]
"categories": ["HackTheBox OSCP-Like"]
"author": "Ghostbyt3"
"description": "We are going to pwn Networked from Hack The Box."
"featured_image": "/img/htb-networked/1.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox OSCP-Like - Writeup"
---


![Untitled](/img/htb-networked/1.png)

We are going to pwn Networked from Hack The Box.

Link : [https://www.hackthebox.eu/home/machines/profile/203](https://www.hackthebox.eu/home/machines/profile/203)


Like always begin with our Nmap Scan.

## Nmap Scan Results

```bash
PORT    STATE  SERVICE
22/tcp  open   ssh
80/tcp  open   http
443/tcp closed https


PORT    STATE  SERVICE VERSION
22/tcp  open   ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 22:75:d7:a7:4f:81:a7:af:52:66:e5:27:44:b1:01:5b (RSA)
|   256 2d:63:28:fc:a2:99:c7:d4:35:b9:45:9a:4b:38:f9:c8 (ECDSA)
|_  256 73:cd:a0:5b:84:10:7d:a7:1c:7c:61:1d:f5:54:cf:c4 (ED25519)
80/tcp  open   http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
443/tcp closed https
Aggressive OS guesses: Linux 3.10 - 4.11 (94%), Linux 3.18 (91%), Linux 3.2 - 4.9 (91%), Linux 3.16 (90%), Linux 4.4 (90%), Crestron XPanel control system (89%), OpenWrt Kamikaze 7.09 (Linux 2.6.22) (89%), Linux 3.11 - 3.12 (89%), HP P2000 G3 NAS device (89%), Linux 3.13 (88%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
```

## HTTP Enumeration

Lets Begin with HTTP

![Untitled](/img/htb-networked/2.png)

Looks like a normal webpage , run our Gobuster to get any useful directories.

```bash
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.146
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/01/03 09:31:00 Starting gobuster
===============================================================
/.htpasswd (Status: 403)
/.hta (Status: 403)
/.htaccess (Status: 403)
/backup (Status: 301)
/cgi-bin/ (Status: 403)
/index.php (Status: 200)
/uploads (Status: 301)
===============================================================
2020/01/03 09:33:22 Finished
===============================================================
```

While Checking ``/backup`` found ``backup.tar``[br/](br/)
![Untitled](/img/htb-networked/3.png)

So I dowloaded it to my machine , to see whats inside.[br/](br/)
![Untitled](/img/htb-networked/4.png)[br/](br/)
It has some ``php`` file maybe webpages of the machine

While seeing the code ``upload.php`` as some filter process. It accepts only jpg,png,gif,jpeg.[br/](br/)
![Untitled](/img/htb-networked/5.png)

### File Upload Vulnerability

``Upload.php``[br/](br/)
![Untitled](/img/htb-networked/6.png)

``photos.php``[br/](br/)
![Untitled](/img/htb-networked/7.png)
Looks like we can view uploaded files in ``photos.php``.

So I created my payload [br/](br/)
![Untitled](/img/htb-networked/8.png)[br/](br/)
By adding ``GIF89`` on the first it makes the payload as ``.gif`` [br/](br/)
![Untitled](/img/htb-networked/9.png)

I uploaded my payload in ``upload.php``[br/](br/)
![Untitled](/img/htb-networked/10.png)

Started my listener and reloaded ``photos.php``[br/](br/)
![Untitled](/img/htb-networked/11.png)[br/](br/)
I got the shell!!

## Getting User Shell


While Checking Home directory there is a file called ``check_attack.php``[br/](br/)
![Untitled](/img/htb-networked/12.png)

While reading it I came to know it executes ``nohup``

>  nohup - run a command immune to hangups, with output to a non-tty

And ``nohup`` do force ``rm`` all files($value) in the folder ($path).

```
rm - remove files or directories
-f - force
```

The path is defined as ``var/www/html/uploads``.
So we need to hijack it.

![Untitled](/img/htb-networked/13.png)

I created a file that hold payload in its name. By using semicolon in the beginning it makes the ``nohup`` to end after it ``rm`` and then exec ``nc``.

Started my listener

![Untitled](/img/htb-networked/14.png)

I got the user shell!!

## Privilege Escalation

As always I started with ``sudo -l``

![Untitled](/img/htb-networked/15.png)

We can run ``changename.sh`` as root without password.
So I tested to see how it works.

![Untitled](/img/htb-networked/16.png)[br/](br/)
It Looks like some network-scripts 

While searching I got this 

>https://vulmon.com/exploitdetails?qidtp=maillist_fulldisclosure&qid=e026a0c5f83df4fd532442e1324ffa4f

![Untitled](/img/htb-networked/17.png)

By doing so I injected ``/bin/bash`` with ``NAME``
And I got the Root Shell


 

