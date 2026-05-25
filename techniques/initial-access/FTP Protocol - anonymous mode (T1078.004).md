# File Transfer Protocol
File Transfer Protocol - used for communication between user and server (uploading and downloading files; remote connection). Uses **Client-Server Architecture**. By default FTP does not encrypts traffic! so it can be easily intercepted via network sniffing
### Ports: 
**21** - command execution **(TCP 21, Control Channel)**
**20** - data storing **(TCP 20, Data Channel)**

### Vulnerability Description / Initial Access
Misconfiguration of FTP allowing an attacker to login in ghost mode, 
using just username "anonymous" without any password.

``` bash
Nmap scan report for {target_IP}
Host is up (0.21s latency).
Not shown: 998 closed ports

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 ftp ftp        33 Jun 08 10:58 allowed.userlist
| -rw-r--r--    1 ftp ftp        62 Apr 20 11:32 allowed.userlist.passwd
| ftp-syst:
|   STAT:
| FTP server status:
|     Connected to ::ffff:{user_IP}
|     Logged in as ftp
|     TYPE: ASCII
|     No session bandwidth limit
|     Session timeout in seconds is 300
|     Control connection is plain text
|     Data connections will be plain text
|     At session startup, client count was 2
|     vsFTPd 3.0.3 - secure, fast, stable
|_End of status
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Smash - Bootstrap Business Template
Service Info: OS: Unix
```

#### FTP cheat sheet
``` bash
$ ftp {target_IP}
Connected to {target_IP}.
220 (vsFTPd 3.0.3)
Name ({target_IP}:{username}): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.

ftp> ls               #list all files
ftp> dir              #same as ls
ftp> cd               #change directory
ftp> pwd              #print current directory
ftp> get file         #download file
ftp> mget *           #download multiple files
ftp> put file         #upload file
ftp> mput *           #upload multiple files
ftp> binary           #binary transger mode
ftp> ascii            #text transfer mode
ftp> status           #session's status
ftp> help             #command list
ftp> quit / bye       #exit from the session
```