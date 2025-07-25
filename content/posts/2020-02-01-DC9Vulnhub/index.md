---
"title": "Vulnhub - DC 9"
"date": 2020-02-01
"tags": ["medium", "sqli", "lfi", "knockd", "hydra", "sudo"]
"keywords": ["medium", "sqli", "lfi", "knockd", "hydra", "sudo"]
"author": "Ghostbyt3"
"description": "Gobuster Scan doesn't give anything useful."
"featured_image": "/img/dc9/1.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "Writeup"
---


Today, We are going to pwn DC 9 by DCAU7 from Vulnhub.

## Description:
DC-9 is another purposely built vulnerable lab with the intent of gaining experience in the world of penetration testing.
The ultimate goal of this challenge is to get root and to read the one and only flag.
Linux skills and familiarity with the Linux command line are a must, as is some experience with basic penetration testing tools.
For beginners, Google can be of great assistance, but you can always tweet me at @DCAU7 for assistance to get you going again. But take note: I won't give you the answer, instead, I'll give you an idea about how to move forward.


Download Link : [https://www.vulnhub.com/entry/dc-9,412/](https://www.vulnhub.com/entry/dc-9,412/)

Lets Begin with our nmap scan as always

## Nmap Scan Results:
```
PORT   STATE    SERVICE
22/tcp filtered ssh
80/tcp open     http

PORT   STATE    SERVICE VERSION
22/tcp filtered ssh
80/tcp open     http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Example.com - Staff Details - Welcome
MAC Address: 08:00:27:1C:9A:EF (Oracle VirtualBox virtual NIC)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop

```

`` Note : SSH is Filtered ``

Let's Begin with Webpage[br/](br/)
![Untitled](/img/dc9/1.png)

Gobuster Scan doesn't give anything useful.

While checking the webpages found 2 interesting pages.

``/search.php``[br/](br/)
![Untitled](/img/dc9/2.png)[br/](br/)
In ``search.php`` we can search the staff name and It will display their details. ``display.php`` contains all staff details.

``/manage.php``[br/](br/)
![Untitled](/img/dc9/3.png)

I tried normal sql injection commands on ``/manage.php`` nothing worked.

So I capture ``/search.php`` request and try that with ``sqlmap``

> sqlmap - automatic SQL injection tool

![Untitled](/img/dc9/4.png)[br/](br/)

![Untitled](/img/dc9/5.png)
![Untitled](/img/dc9/6.png)

Yes It worked! Got 3 available databases.

```
available databases [3]:
[*] information_schema
[*] Staff
[*] users
```
So I tried to see whats in ``users``

![Untitled](/img/dc9/7.png)
![Untitled](/img/dc9/8.png)

```
-D DB               DBMS database to enumerate
--tables            Enumerate DBMS database tables
```
Now I got the table.

Lets dig inside it![br/](br/)
![Untitled](/img/dc9/9.png)[br/](br/)
![Untitled](/img/dc9/10.png)

`` -T TBL              DBMS database table(s) to enumerate ``

Lets dump them[br/](br/)
![Untitled](/img/dc9/11.png)[br/](br/)
![Untitled](/img/dc9/12.png)

We got the users and their passwords!

```
Database: users
Table: UserDetails
[17 entries]
+----+------------+---------------+---------------------+-----------+-----------+
| id | lastname   | password      | reg_date            | username  | firstname |
+----+------------+---------------+---------------------+-----------+-----------+
| 1  | Moe        | 3kfs86sfd     | 2019-12-29 16:58:26 | marym     | Mary      |
| 2  | Dooley     | 468sfdfsd2    | 2019-12-29 16:58:26 | julied    | Julie     |
| 3  | Flintstone | 4sfd87sfd1    | 2019-12-29 16:58:26 | fredf     | Fred      |
| 4  | Rubble     | RocksOff      | 2019-12-29 16:58:26 | barneyr   | Barney    |
| 5  | Cat        | TC&TheBoyz    | 2019-12-29 16:58:26 | tomc      | Tom       |
| 6  | Mouse      | B8m#48sd      | 2019-12-29 16:58:26 | jerrym    | Jerry     |
| 7  | Flintstone | Pebbles       | 2019-12-29 16:58:26 | wilmaf    | Wilma     |
| 8  | Rubble     | BamBam01      | 2019-12-29 16:58:26 | bettyr    | Betty     |
| 9  | Bing       | UrAG0D!       | 2019-12-29 16:58:26 | chandlerb | Chandler  |
| 10 | Tribbiani  | Passw0rd      | 2019-12-29 16:58:26 | joeyt     | Joey      |
| 11 | Green      | yN72#dsd      | 2019-12-29 16:58:26 | rachelg   | Rachel    |
| 12 | Geller     | ILoveRachel   | 2019-12-29 16:58:26 | rossg     | Ross      |
| 13 | Geller     | 3248dsds7s    | 2019-12-29 16:58:26 | monicag   | Monica    |
| 14 | Buffay     | smellycats    | 2019-12-29 16:58:26 | phoebeb   | Phoebe    |
| 15 | McScoots   | YR3BVxxxw87   | 2019-12-29 16:58:26 | scoots    | Scooter   |
| 16 | Trump      | Ilovepeepee   | 2019-12-29 16:58:26 | janitor   | Donald    |
| 17 | Morrison   | Hawaii-Five-0 | 2019-12-29 16:58:28 | janitor2  | Scott     |
+----+------------+---------------+---------------------+-----------+-----------+
```
I tried login with them in ``/manage.php`` but didn't worked so I moved to another Database.

![Untitled](/img/dc9/13.png)[br/](br/)
![Untitled](/img/dc9/14.png)

```
+--------+----------------------------------+----------+
| UserID | Password                         | Username |
+--------+----------------------------------+----------+
| 1      | 856f5de590ef37314e7c3bdf6f8a66dc | admin    |
+--------+----------------------------------+----------+
```
I cracked them with online crackers

> https://crackstation.net/

![Untitled](/img/dc9/15.png)

Now I tried with these creds `` admin:transorbital1 ``

![Untitled](/img/dc9/16.png)

Successfully Logged in[br/](br/)
![Untitled](/img/dc9/17.png)[br/](br/)
While looking at the webpage there is a message `` File does not exist `` on the footer.

So I tried normal LFI
![Untitled](/img/dc9/18.png)

Looks like its working.I searched for any log files available but nothing.

We already know that SSH port is filtered, it may be because of ``knockd service``.

> Once knockd is installed and running, you modify your firewall rules (e.g. iptables) to drop all incoming traffic to port 22. To the outside world, it's exactly as if you are not running SSH at all.


> Reference : https://www.endpoint.com/blog/2009/11/16/port-knocking-with-knockd

There is a way to bypass it.

> Port Knocking works by opening ports on a firewall by generating a connection attempt on a set of prespecified closed ports. Once a correct sequence of connection attempts is received, the firewall will open the port that was previously closed.

So we need the sequence to open the port.

>https://blog.rapid7.com/2017/10/04/how-to-secure-ssh-server-using-port-knocking-on-ubuntu-linux/

From the blog I came to know we can see the sequence in ``/etc/knockd.conf``

![Untitled](/img/dc9/19.png)[br/](br/)
We got the sequence!!!

According to rapid7 we can use ``telnet`` for port knocking.[br/](br/)
![Untitled](/img/dc9/20.png)

We opened the ``ssh`` port!

Now I can login , I tried with the same ``admin`` creds , didn't worked but we already have staffs name and their passwords.
So I started brute forcing using ``hydra``. 

> hydra - a very fast network logon cracker which supports many different services.

![Untitled](/img/dc9/21.png)

Got 3 user name and password. I logged in with `` janitor : Ilovepeepee ``[br/](br/)
![Untitled](/img/dc9/22.png)[br/](br/)
While checking the home directory found a hidden folder[br/](br/)
![Untitled](/img/dc9/23.png)[br/](br/)
It gives us some new password so I added them into our old list and start bruteforcing again.

![Untitled](/img/dc9/24.png)
Now I got one more user password `` fredf : B4-Tru3-001 ``
![Untitled](/img/dc9/25.png)

## Privilege Escalation:

![Untitled](/img/dc9/26.png)

Started with ``sudo -l`` like always. And seems like I can run ``test`` as root without password.
When I tried executing it shows some message `` read append ``. So I check the directory and found a py script.
The Script take the arguments that we give and open and write on the file.

So I created my own password

![Untitled](/img/dc9/27.png)

> openssl passwd -1

![Untitled](/img/dc9/28.png)

> w0lf:$1$AtYW6q22$t3BQ9bt9XrTfe.oK9SPpG/:0:0:w0lf:/root:/bin/bash

![Untitled](/img/dc9/29.png)

Im Root!

Got the Flag

![Untitled](/img/dc9/30.png)









