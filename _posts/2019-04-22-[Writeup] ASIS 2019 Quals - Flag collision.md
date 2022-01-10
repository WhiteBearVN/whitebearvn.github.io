---
title : "[Writeup] ASIS 2019 Quals - Flag collision"
author : "F4c3l3ss_"
date : 2019-04-22
tags : ["ASIS","writeup"]
categories: [Writeup, ASIS2019]
---

In this challenge, we need to sumbit two string differene but same length and same crc 32. After I try to brute force two string with length is 15 and submit to server, I received the example of admin is two strings:  ASIS{4LEVv9no8} and ASIS{wpQ78d6lk}. These things help me alot becasue in next stages, server will request random length of 2 strings but same crc32. 

After do some stuff, I realized if we add some stuff to that strings the crc32 still same.

For example:

> CRC32(ASIS{wpQ78d6lkBBB}) and CRC32(ASIS{4LEVv9no8BBB}) is same.

Ok so we just need to add "B" word to strings and send to sever and take the flag
My code:

```python
import hashlib
import itertools
from pwn import *
import sys
def md5(pow_target):
    pow = ""
    for word in itertools.product(string.printable, repeat=5):
        if hashlib.md5(''.join(word)).hexdigest()[-6:] == pow_target:
            pow = ''.join(word)
            return pow
def sha1(pow_target):
    pow = ""
    for word in itertools.product(string.printable, repeat=5):
        if hashlib.sha1(''.join(word)).hexdigest()[-6:] == pow_target:
            pow = ''.join(word)
            return pow
def sha224(pow_target):
    pow = ""
    for word in itertools.product(string.printable, repeat=5):
        if hashlib.sha224(''.join(word)).hexdigest()[-6:] == pow_target:
            pow = ''.join(word)
            return pow
def sha256(pow_target):
    pow = ""
    for word in itertools.product(string.printable, repeat=5):
        if hashlib.sha256(''.join(word)).hexdigest()[-6:] == pow_target:
            pow = ''.join(word)
            return pow
def sha384(pow_target):
    pow = ""
    for word in itertools.product(string.printable, repeat=5):
        if hashlib.sha384(''.join(word)).hexdigest()[-6:] == pow_target:
            pow = ''.join(word)
            return pow
def sha512(pow_target):
    pow = ""
    for word in itertools.product(string.printable, repeat=5):
        if hashlib.sha512(''.join(word)).hexdigest()[-6:] == pow_target:
            pow = ''.join(word)
            return pow
host = '37.139.9.232'
port = 19199
r = remote(host,port)
msg = r.recvuntil('\n')
#print msg
msg = msg.split(' ')
#print msg
pow_target = msg[-1].strip()
if 'md5' in msg[-3]:
    payload = md5(pow_target)
elif 'sha1' in msg[-3]:
    payload = sha1(pow_target)
elif 'sha224' in msg[-3]:
    payload = sha1(pow_target)
elif 'sha256' in msg[-3]:
    payload = sha256(pow_target)
elif 'sha384' in msg[-3]:
    payload = sha384(pow_target)
elif 'sha512' in msg[-3]:
    payload = sha512(pow_target)
else:
    print msg[-3]
r.sendline(payload)
#print r.recv(1024)
print 'stage1'

r.sendline('ASIS{wpQ78d6lk}, ASIS{4LEVv9no8}')
#print r.recv(1024)
print 'stage2'
i=0
while 1:
    a = 'ASIS{4LEVv9no8'
    b = 'ASIS{wpQ78d6lk'
    if i == 14:
        print r.recv(1024)
        sys.exit()
    r.recvuntil(':)\n')
    msg = r.recvuntil(':|\n')
    msg = msg.split(' ')
    number = int(msg[10])
    repeat = number-len(a)-1
    a = a + "b" * repeat + "}"
    b = b + "b" * repeat + "}"
    m = str(a)+', '+str(b)
    m = str(m)
    r.sendline(m)
    i+=1
```

And after 14 times, I got a flag:

![IMG](/assets/img/blog/asis2019.PNG)





