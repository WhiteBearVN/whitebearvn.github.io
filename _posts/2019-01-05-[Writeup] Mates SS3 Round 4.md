---
title: "[Writeup] Mates SS3 Round 4"
author: F4c3l3ss_
date: 2019-01-05
categories: [Writeup,MatesCTF]
tags: [writeup,forensics,ctf,programing]
math: true
mermaid: true
---

## Programing
Đề bài làm bắt chúng ta phải làm sao từ 2 bình nước x, y có thể tích vx, vy và phải đong nước làm sao cho được z lít. Đây là bài toàn đong nước kinh điển, may mắn là mình đã được học qua ở trường nên có thể làm được bài này một cách nhanh chóng bằng thuật toán dẫn luật.

Soure code:

```
Luat 1: Neu VX day thi do nuoc trong binh X di               
Luat 2: Neu VY rong thi do day nuoc cho binh Y
Luat 3: Neu VX khong day va VY khong rong thi 
trut nuoc tu binh Y sang binh X cho den khi binh X day    

e lam rong binh
f lam day binh
o do tu binh nay sang binh kia

```
```python
from pwn import *
host = '125.235.240.166'

port =  11223
r = remote(host,port)
def dongnuoc(Vx,Vy,z):
    x = 0
    y = 0
    s = ''
    s = ""
    while (x!=z) and (y!=z):
        if x == Vx:
            x = 0
            s += '1:e_'
        if y == 0:
            y = Vy
            s += '2:f_'
        if y > 0:
            k = min(Vx-x,y)             
            x = x+k
            y = y-k 
            s += '2:o_'
    return s

def agr():
    r.recvuntil('1: ')
    Vx = int(r.recvuntil('\n').strip())
    r.recvuntil('2: ')
    Vy = int(r.recvuntil('\n').strip())
    r.recvuntil('z: ')
    z = int(r.recvuntil('\n').strip())
    return Vx,Vy,z

for i in range(5):   
    Vx,Vy,z = agr()
    s = dongnuoc(Vx,Vy,z)
    s = s[:-1]
    r.recvuntil('op> ')
    r.sendline(s)
    print r.recvuntil('\n')
    print r.recvuntil('\n')
r.interactive()
```

### Forensics
Đề bài là một file pcap với giao thức chính là NTP và data strem là một chuỗi base64 nhưng không decode ra gì cả, sau khi xem tổng thể mình để ý thấy có 1 gói tin Malformed packet tra google mình biết đó là PNG bị hỏng trong quá trình chuyển tin. Để ý lại một lần nữa mình mới thấy là bản thân khá fail khi chỉ loại 1 loạt dấu !!! mà không thử loại loạt chữ AAA ở đầu. Thử decdoe bằng đoạn mới mình có một bức ảnh PNG chứa flag:

![IMG](/assets/img/blog/matesss3r4.jpg)
