---
title : "[Writeup] Hack The Box - Open Admin"
author : "F4c3l3ss_"
date : 2020-05-09
tags : ["hackthebox","writeup"]
categories: [Writeup, hackthebox]
---
![IMG](/assets/img/blog/1_openadmin.png)

The way how I took user and root of this machine.

## 0x01: Looking

Access the address of this machine I only can see the default index page of Apache2, so I used ```dirb``` & ```nmap``` to scan and knows more about this machine.

![IMG](/assets/img/blog/2_dirb_result.png)
_Dirb result_

![IMG](/assets/img/blog/3_nmap.png)
_Nmap result_

Access ```/music``` I see the login function, and it redirect me to ```/ona```. 

## 0x02: Access shell

![IMG](/assets/img/blog/4_opennet.png)
_OpenNetAdmin_

After googling, I found RCE vulnerability, you can download the payload [here.](https://www.exploit-db.com/exploits/47691).

![IMG](/assets/img/blog/5_oash.png)

After cat ```/etc/passwd```, I realize I may need to use the privilege escalation technique from ```www-data``` to 2 users ```jimmy``` and ```joanna``` before taking a ```root```.

## 0x03: Access user 1

Searching in the main folder of ona, I found the password to login one of the users in the database.

```bash
$ cat local/config/database_settings.inc.php
```

![IMG](/assets/img/blog/6_php.png)

Tried on both users ```jimmy``` & ```jonna``` I can logged-in with the username ```jimmy``` via ssh with its password.

## 0x04: Access user 2

Searching in the machine I found in ```/var/www``` there is a directory ```internal``` with ```jimmy``` permission.

After looking at the php source code, the workflow is  ```index.php``` >>> ```main.php``` and read the RSA key, but it can’t read because it does not have any permission.

Use command ```netstat -natpe``` I realized that the php source code is running at ```root``` because the user id is 0. Because of that, I just need to access  ```/main.php``` and take the RSA private key. 

```bash
$ curl 127.0.0.1:52846/main.php 
```
![IMG](/assets/img/blog/7_mainphp.png)
_RSA private key_

But it is not the end yet, I still need to find the passphrase of this private key to access via ssh.

Use ```john``` and ```rockyou``` wordlist:

```bash
# python ssh2john.py joanna_pri_key > joanna_pri_key.hash
# john --wordlist=/root/Downloads/rockyou.txt --format=SSH joanna_pri_key.hash
```

And the passphrase is: bloodninjas

## 0x05: Access root

At /opt , we have a one file priv and it's permission is 

```bash
352 -rw-r - r - 1 root root 0 Nov 22 23:49 priv
```

Beside that, I also used sudo -l and knew that nano can be used under root permission without password. It means that I can use GTFOBins to privilege escalation to root.

```bash
$ sudo nano priv
^R^X
reset; sh 1>&0 2>&0
```

![IMG](/assets/img/blog/8_root.png)
_Got root_

And tada, I got root! That is how I own the user and root of this machine. Hope you guys learned something new from this machine.

Cheers! 👍

P/S: Thank you GinaTU aka Tứ Diệp Thảo to help me edit this post

