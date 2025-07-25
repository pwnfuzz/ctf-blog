---
"title": "Vulnhub - DC 8"
"date": 2019-11-26
"tags": ["medium", "drupal", "sqli"]
"keywords": ["medium", "drupal", "sqli"]
"categories": "Vulnhub"
"author": "Ghostbyt3"
"description": "Today, We are going to pwn DC 8 by DCAU7 from Vulnhub"
"featured_image": "/img/dc8/1.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "Vulnhub - Writeup"
---


![Untitled](/img/dc8/1.png)
Today, We are going to pwn DC 8 by DCAU7 from Vulnhub


## Description
>DC-8 is another purposely built vulnerable lab with the intent of gaining experience in the world of penetration testing.
This challenge is a bit of a hybrid between being an actual  challenge, and being a "proof of concept" as to whether two-factor  authentication installed and configured on Linux can prevent the Linux  server from being exploited.
The "proof of concept" portion of this challenge eventuated as a  result of a question being asked about two-factor authentication and  Linux on Twitter, and also due to a suggestion by @theart42.
The ultimate goal of this challenge is to bypass two-factor authentication, get root and to read the one and only flag.
You probably wouldn't even know that two-factor authentication was  installed and configured unless you attempt to login via SSH, but it's  definitely there and doing it's job.
Linux skills and familiarity with the Linux command line are a must, as is some experience with basic penetration testing tools.
For beginners, Google can be of great assistance, but you can always  tweet me at @DCAU7 for assistance to get you going again. But take note:  I won't give you the answer, instead, I'll give you an idea about how  to move forward.

Download Link : [https://www.vulnhub.com/entry/dc-8,367/](https://www.vulnhub.com/entry/dc-8,367/)

Lets Begin our Nmap Scan

## Nmap Scan Results
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u1 (protocol 2.0)
| ssh-hostkey: 
|   2048 35:a7:e6:c4:a8:3c:63:1d:e1:c0:ca:a3:66:bc:88:bf (RSA)
|   256 ab:ef:9f:69:ac:ea:54:c6:8c:61:55:49:0a:e7:aa:d9 (ECDSA)
|_  256 7a:b2:c6:87:ec:93:76:d4:ea:59:4b:1b:c6:e8:73:f2 (ED25519)
80/tcp open  http    Apache httpd
|_http-generator: Drupal 7 (http://drupal.org)
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/ 
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
|_/LICENSE.txt /MAINTAINERS.txt
|_http-server-header: Apache
|_http-title: Welcome to DC-8 | DC-8
MAC Address: 08:00:27:4B:B7:5B (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## HTTP

![Untitled](/img/dc8/1.png)

It is a ``DrupalCMS``

While checking the webpage , this look odd ``/?nid=``

![Untitled](/img/dc8/2.png)

So I tried passing some other id's and it shows some SQL error
![Untitled](/img/dc8/3.png)

## SQL Injection

Lets run our SQLmap

![Untitled](/img/dc8/4.png)[br/](br/)
![Untitled](/img/dc8/5.png)

It showed us that there are 2 available databases in the target machine which are:

```
1.d7db
2.information_schema
```

Lets see whats in the database of ``d7db``

![Untitled](/img/dc8/6.png)[br/](br/)
![Untitled](/img/dc8/7.png)

```
-D DB               DBMS database to enumerate
--tables            Enumerate DBMS database tables
```
We found a table named ‘users’.

![Untitled](/img/dc8/8.png)[br/](br/)
![Untitled](/img/dc8/9.png)

`` -T TBL              DBMS database table(s) to enumerate ``

It looks like there are some name and pass so lets dump them!.

![Untitled](/img/dc8/10.png)[br/](br/)
![Untitled](/img/dc8/11.png)

We found users and hashes 
> Users - admin,john

Lets try crack them using ``john``

![Untitled](/img/dc8/12.png)

We found a password ``turtle``

I logged in with ``john:turtle``[br/](br/)
![Untitled](/img/dc8/13.png)

## Getting Shell

Now its time to get Reverse shell. I figure out that we need to edit the ``form-setting`` and change the ``text format`` to ``php code``
![Untitled](/img/dc8/14.png)

Once I submit[br/](br/)
![Untitled](/img/dc8/15.png)

Listening on my machine , I got the shell[br/](br/)
![Untitled](/img/dc8/16.png)

## Privilege Escalation

I uploaded my Linux Enumeration Script 
Found some SETUID Bit
![Untitled](/img/dc8/17.png)[br/](br/)
Where ``exim4`` is unusual
 
Searchsploit shows lot of exploits [br/](br/)
![Untitled](/img/dc8/18.png)

so lets check its version[br/](br/)
![Untitled](/img/dc8/19.png)

And Found an exploit for that version![br/](br/)
>https://www.exploit-db.com/exploits/46996

I uploaded it in the victim machine and tried running it but it shows some ``bad interpreter`` so i googled about it

![Untitled](/img/dc8/20.png)

Lets try this 

![Untitled](/img/dc8/21.png)

>sed -i -e 's/\r$//' script.sh

It worked 

![Untitled](/img/dc8/22.png)


We Got Root!


