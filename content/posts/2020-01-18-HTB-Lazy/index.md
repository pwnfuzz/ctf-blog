---
"title": "Hack The Box - Lazy"
"date": 2020-01-18
"tags": ["linux", "medium", "path", "padbuster"]
"keywords": ["linux", "medium", "path", "padbuster"]
"author": "Ghostbyt3"
"description": "We are going to pwn Lazy from Hack The Box."
"featured_image": "/img/htb-lazy/1.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "Writeup"
---


![Untitled](/img/htb-lazy/1.png)

We are going to pwn Lazy from Hack The Box.

Link : [https://www.hackthebox.eu/home/machines/profile/18](https://www.hackthebox.eu/home/machines/profile/18)


Lets Begin with our Initial Nmap Scan.

Nmap Scan Results:

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http


PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 e1:92:1b:48:f8:9b:63:96:d4:e5:7a:40:5f:a4:c8:33 (DSA)
|   2048 af:a0:0f:26:cd:1a:b5:1f:a7:ec:40:94:ef:3c:81:5f (RSA)
|   256 11:a3:2f:25:73:67:af:70:18:56:fe:a2:e3:54:81:e8 (ECDSA)
|_  256 96:81:9c:f4:b7:bc:1a:73:05:ea:ba:41:35:a4:66:b7 (ED25519)
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: CompanyDev
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.12 (95%), Linux 3.13 (95%), Linux 3.16 (95%), Linux 3.2 - 4.9 (95%), Linux 3.8 - 3.11 (95%), Linux 4.8 (95%), Linux 4.4 (95%), Linux 4.9 (95%), Linux 3.18 (95%), Linux 4.2 (95%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## HTTP:

![Untitled](/img/htb-lazy/2.png)
There is a ``Login and Register`` tab. I created a new account.
![Untitled](/img/htb-lazy/3.png)[br/](br/)
Once Account Created, I logged in with the credentials.
![Untitled](/img/htb-lazy/4.png)

I intercept the webpage using ``burp`` to check if there is anything suspecious.
![Untitled](/img/htb-lazy/5.png)
And I found there is an ``auth`` in Cookie.

So I changed the ``auth`` and ``send`` that request and is shows ``Invalid Padding``
![Untitled](/img/htb-lazy/6.png)
This make me think of a popular attack name ``Padding Oracle Attack``

>Padding oracle attack is an attack which uses the padding validation of a cryptographic message to decrypt the ciphertext. In cryptography, variable-length plaintext messages often have to be padded (expanded) to be compatible with the underlying cryptographic primitive. The attack relies on having a "padding oracle" who freely responds to queries about whether a message is correctly padded or not.

### Reference:
>https://blog.gdssecurity.com/labs/2010/9/14/automated-padding-oracle-attacks-with-padbuster.html

We can use the tool ``Padbuster``

This is the format 
```
 padbuster URL EncryptedSample BlockSize [options]

The default block size is 8k in Oracle. This is the most common. Sometimes, people create the database with 16k block size for datawarehouses. You can also find some 32k block size, but less common which means more bug

-cookies [HTTP Cookies]: Cookies (name1=value1; name2=value2)
```
![Untitled](/img/htb-lazy/7.png)[br/](br/)
I used my auth from the cookie.[br/](br/)
![Untitled](/img/htb-lazy/8.png)

We can change the ``user=admin`` so we can get his ``auth`` and login as ``admin``.
![Untitled](/img/htb-lazy/9.png)[br/](br/)
![Untitled](/img/htb-lazy/10.png)[br/](br/)
We got ``admin`` auth.

So I replaced our ``auth`` with the admin's auth.
![Untitled](/img/htb-lazy/11.png)

Yeah We logged in as admin, There is ``My Key`` which gives us ssh private key.
![Untitled](/img/htb-lazy/12.png)

Downloaded to my machine and gave it permission and I used that key to login as ``mitsos`` because its the file name.
![Untitled](/img/htb-lazy/13.png)

## Privilege Escalation:

There is a binary file called ``backup`` when I execute it prints us ``/etc/shadow``[br/](br/)
![Untitled](/img/htb-lazy/14.png)

While checking the ``strings`` of it[br/](br/)
![Untitled](/img/htb-lazy/15.png)

It shows ``cat /etc/shadow`` and ``cat`` full path is not specified.

So I checked ``PATH`` and where is ``cat`` actually located.[br/](br/)
![Untitled](/img/htb-lazy/16.png)

``cat`` is in ``/bin/`` but we know that the ``PATH`` first search in ``/usr/local/sbin`` so we can create ``cat`` file with a reverse shell and place it in ``/usr/local/sbin``
But we dont have write permission in ``/usr/local/sbin``

I created ``cat`` file in ``/tmp`` and give execute permission.[br/](br/)
![Untitled](/img/htb-lazy/17.png)

This will give us shell if its executed.[br/](br/)
![Untitled](/img/htb-lazy/18.png)

We don't have permission on ``/usr/local/sbin`` so I changed the ``/tmp`` in  ``PATH``  to make it search there first.

> export PATH=/tmp:$PATH

![Untitled](/img/htb-lazy/19.png)

Now If I execute the ``backup`` binary it searches for the ``cat`` in ``/tmp`` first and once its found it executes and give me shell as root.
![Untitled](/img/htb-lazy/20.png)

