---
"title": "Hack The Box - OpenAdmin"
"date": 2020-05-02
"tags": ["linux", "easy", "sudo", "port_forward"]
"keywords": ["linux", "easy", "sudo", "port_forward"]
"categories": "HackTheBox"
"author": "Ghostbyt3"
"description": "We are going to pwn OpenAdmin from Hack The Box."
"featured_image": "/img/htb-openadmin/1.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox - Writeup"
---


![Untitled](/img/htb-openadmin/1.png)

We are going to pwn OpenAdmin from Hack The Box.

Link: [https://www.hackthebox.eu/home/machines/profile/222](https://www.hackthebox.eu/home/machines/profile/222)

Like always begin with our Nmap Scan.

## Nmap Scan Results

```bash
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4b:98:df:85:d1:7e:f0:3d:da:48:cd:bc:92:00:b7:54 (RSA)
|   256 dc:eb:3d:c9:44:d1:18:b1:22:b4:cf:de:bd:6c:7a:54 (ECDSA)
|_  256 dc:ad:ca:3c:11:31:5b:6f:e6:a4:89:34:7c:9b:e5:50 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.2 - 4.9 (95%), Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), Linux 3.18 (94%), Linux 3.16 (93%), ASUS RT-N56U WAP (Linux 3.4) (93%), Oracle VM Server 3.4.2 (Linux 4.1) (93%), Android 4.1.1 (93%), Android 4.2.2 (Linux 3.4) (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
```

## HTTP Enumeration

While checking the webpage, its a default apache page.

![Untitled](/img/htb-openadmin/2.png)

So I started my Gobuster to get any interesting directories.

### Gobuster Result

```bash
root@kali:~# gobuster dir -u 10.10.10.171 -w /usr/share/wordlists/dirb/common.txt 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.171
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/01/06 19:28:04 Starting gobuster
===============================================================
/artwork (Status: 301)
/index.html (Status: 200)
/music (Status: 301)
/server-status (Status: 403)
===============================================================
2020/01/06 19:29:57 Finished
===============================================================
```

While checking the pages, In `/music` there is an login button.

![Untitled](/img/htb-openadmin/3.png)

When I click that it redirects me too `/ona`

![Untitled](/img/htb-openadmin/4.png)

It's an `OpenNetAdmin` dashboard and we have its version now.

So I started looking for exploits for that version.

![Untitled](/img/htb-openadmin/5.png)

I tried the RCE (Remote Code Execution).

>[https://www.exploit-db.com/exploits/47691](https://www.exploit-db.com/exploits/47691)

![Untitled](/img/htb-openadmin/6.png)

It works It shows me all the files in the `/opt/ona/www/` folder so we are in the web server directory.
Since the script is limited I can't move to any other directory as this is only using single web requests and I can't get any reverse shell.

![Untitled](/img/htb-openadmin/7.png)

So I uploaded [p0wny](https://github.com/flozz/p0wny-shell) to the box by starting Python HTTP server in my machine and using wget I uploaded the webshell to the box.

![Untitled](/img/htb-openadmin/7.1.png)

And I accessed that from the webpage.

## Getting reverse shell from Webshell

![Untitled](/img/htb-openadmin/8.png)

I Found 2 users `jimmy` and `joanna` and  I got a reverse shell from here.

While checking the directories found `/local/config/database_settings.inc.php`

![Untitled](/img/htb-openadmin/9.png)

Since its Mysql Credentials, I Tried login with `mysql` but failed, Later I found the password works for `jimmy` via `ssh`

## User Jimmy

![Untitled](/img/htb-openadmin/10.png)

`jimmy : n1nj4W4rri0R!`

I found a folder `/var/www/internal` which contains some interesting files.

![Untitled](/img/htb-openadmin/11.png)

`index.php`

![Untitled](/img/htb-openadmin/12.png)

It gives us `jimmy` hash, I cracked it in [Crackstation](https://crackstation.net/).

![Untitled](/img/htb-openadmin/13.png)

It gives me a new password `Revealed`

`Main.php`

![Untitled](/img/htb-openadmin/14.png)

So according to the files we found in `/var/www/internal/`, if we logged in `/index.php` with the credentials it directs to `/main.php` and it gives us `joanna` private key but I can't found any other login pages.

I uploaded my Linux Enumeration Script and found there is port `52846` listening on the machine.

![Untitled](/img/htb-openadmin/15.png)

Port `3306` is the default port for the `MySQL` Protocol ( port ), which is used by the mysql client, MySQL Connectors, and utilities such as mysqldump and mysqlpump.

So we can `port forward` port `52846` to our local machine and see the webpage

For that, we can use SSH Interactive shell to do Local Port Forward. To open SSH Interactive shell type `~C` and I forwarded the port to `52000`.

![Untitled](/img/htb-openadmin/16.png)

Now I can view the webpage in my machine and here it is the Login page we searched for.

![Untitled](/img/htb-openadmin/17.png)

We already got the creds from `index.php` - `jimmy:Revealed`

![Untitled](/img/htb-openadmin/18.png)

We got the Private key for the user `joanna`.

Converted Privatekey using `ssh2john` so we can crack it with `john` to get the passphrase.
Found the `passphrase` it is `bloodninjas`

![Untitled](/img/htb-openadmin/19.png)

## User Joanna

![Untitled](/img/htb-openadmin/20.png)

## Privilege Escalation

Like always I started with `sudo -l`

![Untitled](/img/htb-openadmin/21.png)

Found that `/bin/nano` can be run as root without password on a file `/opt/priv`.

>[https://gtfobins.github.io/gtfobins/nano/](https://gtfobins.github.io/gtfobins/nano/)

```bash
sudo /bin/nano /opt/priv
^R^X
reset; sh 1>&0 2>&0
```

![Untitled](/img/htb-openadmin/22.png)

Followed the commands 

![Untitled](/img/htb-openadmin/23.png)

I'm Root Now!!