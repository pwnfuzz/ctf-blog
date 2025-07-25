---
"title": "Hack The Box - Traceback"
"date": 2020-08-15
"tags": ["linux.easy", "osint", "cron", "sudo"]
"keywords": ["linux.easy", "osint", "cron", "sudo"]
"categories": ["HackTheBox"]
"author": "Ghostbyt3"
"description": "Traceback is really a good beginner friendly box, getting initial is to look for an existing webshell on the box. There is some sudo stuffs to get user shell and Privesc is by finding a script thats running as root and we have write permission over it."
"featured_image": "/img/htb-traceback/1.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox - Writeup"
---


![Untitled](/img/htb-traceback/1.png)

Traceback is really a good beginner friendly box, getting initial is to look for an existing webshell on the box. There is some sudo stuffs to get user shell and Privesc is by finding a script thats running as root and we have write permission over it. 

Link : [https://www.hackthebox.eu/home/machines/profile/233](https://www.hackthebox.eu/home/machines/profile/233)

Lets Begin with our Initial Nmap Scan.

## Nmap Scan Results

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 96:25:51:8e:6c:83:07:48:ce:11:4b:1f:e5:6d:8a:28 (RSA)
|   256 54:bd:46:71:14:bd:b2:42:a1:b6:b0:2d:94:14:3b:0d (ECDSA)
|_  256 4d:c3:f8:52:b8:85:ec:9c:3e:4d:57:2c:4a:82:fd:86 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Help us
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.80%E=4%D=3/15%OT=22%CT=1%CU=34074%PV=Y%DS=2%DC=T%G=Y%TM=5E6D476
OS:6%P=x86_64-pc-linux-gnu)SEQ(SP=105%GCD=1%ISR=10F%TI=Z%CI=Z%II=I%TS=A)SEQ
OS:(SP=105%GCD=1%ISR=10F%TI=Z%CI=Z%TS=A)OPS(O1=M54DST11NW7%O2=M54DST11NW7%O
OS:3=M54DNNT11NW7%O4=M54DST11NW7%O5=M54DST11NW7%O6=M54DST11)WIN(W1=7120%W2=
OS:7120%W3=7120%W4=7120%W5=7120%W6=7120)ECN(R=Y%DF=Y%T=40%W=7210%O=M54DNNSN
OS:W7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%D
OS:F=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O
OS:=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W
OS:=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%R
OS:IPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)
```

## HTTP Enumeration

This looks an normal webpage. With some message.

![Untitled](/img/htb-traceback/2.png)

While checking the source code of the webpage found a hint.

```html
[center](center)
		[h1](h1)This site has been owned[/h1](/h1)
		[h2](h2)I have left a backdoor for all the net. FREE INTERNETZZZ[/h2](/h2)
		[h3](h3) - Xh4H - [/h3](/h3)
		[!--Some of the best web shells that you might need ;)--](!--Some of the best web shells that you might need ;)--)
	[/center](/center)
```

So I started searching for the best web shells in google with the hint.

![Untitled](/img/htb-traceback/3.png)

The first link is a GitHub repo.

> [https://github.com/TheBinitGhimire/Web-Shells](https://github.com/TheBinitGhimire/Web-Shells)

We need to find which webshell is in the box. `This site has been owned` - Might be an hint that the site owned a webshell.
So I started checking them and found it.

> http://10.10.10.181/smevk.php

![Untitled](/img/htb-traceback/4.png)

Looks like we need Username and Password for login. I checked the github repo of the webshell.

![Untitled](/img/htb-traceback/5.png)

So default password is `admin : admin`

We logged in!!

![Untitled](/img/htb-traceback/6.png)

## Getting Shell as webadmin

Why don't we try to get a reverse shell since we have an upload option here. I uploaded PHP Reverse Shell.

![Untitled](/img/htb-traceback/7.png)

Now open the PHP Reverse shell in one page and started my nc listener. I got the shell as `webadmin`.

![Untitled](/img/htb-traceback/8.png)

Uploaded my Enumeration script and found that we have writeable permission on `/home/webadmin/.ssh/`
So I created a pair of ssh keys in my machine.

![Untitled](/img/htb-traceback/9.png)

I copied the `.pub` key to `/home/sysadmin/.ssh/authorized_keys` so we can connect `webadmin` via ssh.

![Untitled](/img/htb-traceback/10.png)

Logged in via ssh using the private keys as `webadmin`.

![Untitled](/img/htb-traceback/11.png)

## Privilege Escalation from Webadmin to Sysadmin

The first thing I did is `sudo -l` to find that we can run any files as `sysadmin` or `root` without the password.

![Untitled](/img/htb-traceback/12.png)

Looks like `luvit` executable can be run as `sysadmin` without password.

`Note.txt`

```
- sysadmin -
I have left a tool to practice Lua.
I'm sure you know where to find it.
Contact me if you have any question.
```

I think `luvit` might be the `Lua` compiler.
I searched in GTFOBins.

> [https://gtfobins.github.io/gtfobins/lua/](https://gtfobins.github.io/gtfobins/lua/)

```bash
sudo -u sysadmin /home/sysadmin/luvit -e 'os.execute("/bin/sh")'
```

By doing so I got `sysadmin`

![Untitled](/img/htb-traceback/13.png)

## Privilege Escalation to Root

There is `pspy64` in home directory maybe someone uploaded or the creator is giving us hint.
It might be a cronjob running in the background, so I started pspy.

> pspy is a command line tool designed to snoop on processes without need for root permissions. It allows you to see commands run by other users, cron jobs, etc. as they execute. Great for enumeration of Linux systems in CTFs. Also great to demonstrate your colleagues why passing secrets as arguments on the command line is a bad idea.

Whenever we do a new ssh connection. These files running in the background.

![Untitled](/img/htb-traceback/14.png)

When I look at the files in `/etc/update-motd.d/`, I came to know these files are displaying the content when we do ssh login.

```html
sysadmin@traceback:/etc$ ls -l update-motd.d/
total 24
-rwxrwxr-x 1 root sysadmin  981 Mar 15 09:39 00-header
-rwxrwxr-x 1 root sysadmin  982 Mar 15 09:39 10-help-text
-rwxrwxr-x 1 root sysadmin 4264 Mar 15 09:39 50-motd-news
-rwxrwxr-x 1 root sysadmin  604 Mar 15 09:39 80-esm
-rwxrwxr-x 1 root sysadmin  299 Mar 15 09:39 91-release-upgrade
```

And we have writeable permission on them so I edit the `91-release-upgrade` to give us shell when new ssh login.

`vi 91-release-upgrade`

![Untitled](/img/htb-traceback/16.png)

Open 2 terminals - One for new SSH login and another to get a reverse shell.

Note: If you didn't get it in first try 2 or more times because the cron is also replacing the scripts in `/etc/update-motd.d/` by `/var/backups/.update-motd.d/`.

```bash
2020/03/19 21:46:01 CMD: UID=0    PID=2959   | /bin/sh -c sleep 30 ; /bin/cp /var/backups/.update-motd.d/* /etc/update-motd.d/ 
2020/03/19 21:46:01 CMD: UID=0    PID=2958   | /bin/sh -c /bin/cp /var/backups/.update-motd.d/* /etc/update-motd.d/
```

![Untitled](/img/htb-traceback/17.png)

We own the box!