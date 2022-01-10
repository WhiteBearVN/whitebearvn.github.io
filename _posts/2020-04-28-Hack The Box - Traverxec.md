---
title : "[Writeup] Hack The Box - Traverxec"
author : "F4c3l3ss_"
date : 2020-04-28
tags : ["hackthebox","writeup"]
categories: [Writeup, hackthebox]
---
![IMG](/assets/img/blog/0_avarta.png)

The way how I solved Traverxec machine.
## 0x01: Access shell
The first step I always do is using Nmap to scan for open port(s) and service(s):

```bash
$ nmap -sV -sT -sC 10.10.10.165
```
And the output is:
![IMG](/assets/img/blog/1_nmap.png)

Let's check the website:

![IMG](/assets/img/blog/2_website.png)

After checking and taking a look around the website, I realized this is just a static website. Then I googled for nostromo 1.9.6 exploitation, and got RCE vuln at this version. You can use the exploit payload at exploit-db. And let’s access bash:

![IMG](/assets/img/blog/3_accessbash.png)

## 0x02: Access user

Normally, source code of a website is usually saved at ```/var``` directory, I checked with ls command and got the nostromo folder. Going to the conf folder, I found  ```nhttpd.conf```.

![IMG](/assets/img/blog/4_conf.png)

And I think a lot of people will be stuck at this step like me. That is reading .htpasswd then decode it and use it like a ssh password.

But read carefully once again, I realize the homedirs is ```/home```

And ls at ```/home``` only has a david folder. With .htpasswd above, I think maybe ```public_www``` folder can exist inside the david folder.

```bash
$ cd /home/david/public_www
```

And yes, it's worked:

![IMG](/assets/img/blog/5_worked.png)

Let's open ```protected-file-area``` folder
![IMG](/assets/img/blog/6_ls.png)

We have a .tgz file, but can’t extract it because we can only read it. Ok, let’s read it by zcat: Got private_key, let’s find the passphrase with john.


![IMG](/assets/img/blog/7_zcat.png)

```bash
$ python ssh2john.py david_key > david_key.hash
$ john --wordlist=/root/Downloads/rockyou.txt --format=SSH david_key.hash
```
And the passphrase is: ```hunter```

## 0x03: Access root

In ```/home/david/bin``` folder, we have 2 files:

![IMG](/assets/img/blog/8_ls.png)


Let's read ```server-stats.sh```:
![IMG](/assets/img/blog/9_serverstarts.png)
I easily realize that journalctl is running at root.
Looking at GTFOBins,journalctl can used to escalate privileges attack if it's running at root.
The way to trick is make terminal smaller and type ```!/bin/sh```:

![IMG](/assets/img/blog/10_root.png)
Tada!

So that concluded how I solved the machine. Hope that you learned something new with this writeup.

Cheers! 👍

P/S: Thank you GinaTU aka Tứ Diệp Thảo to help me edit this post

