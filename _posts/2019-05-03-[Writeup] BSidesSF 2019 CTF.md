---
title : "[Writeup] BSidesSF 2019 CTF"
author : "F4c3l3ss_"
date : 2019-05-03
tags : ["BSidesSF","writeup"]
categories: [Writeup, "2019"]
---

## table-tennis

Sử dụng wireshark, mình thấy có khá nhiều certificate, tuy nhiên nhưng chứng chỉ này không làm được gì cả. Và số lượng gói tin cũng khá nhiều nên mình vào Statistics -> Conversations để xem các kết nối.

![IMG](/assets/img/blog/BSidesSF2019-1.PNG)

Sau khi filter các kết nối của mạng 172.217.5.100 và 172.217.0.36 mình thấy rằng đó là các gói tin ICMP, request và reply data có đều có chứa mã base64, vì không rành về thư viện scappy lắm và một phần là chuỗi cũng khá ngắn nên mình quyết định là làm bằng tay.

![IMG](/assets/img/blog/BSidesSF2019-2.PNG)

Và chỉ sau chừng 5 phút mình đã có được đoạn base64 hoàn chỉnh:


> Q1RGe0p1c3RBMHV0UDFuUzBuZ0FiZ1Awbmd9

Flag: CTF{JustA0utP1nS0ngAbgP0ng}

## goodlusk1

Ban tổ chức cho mình một file img nên mình dùng Autospy để mở. Bên trong chứa một file LUKS và không có gì khác nên mình extract file đó ra. Google cho mình biết đó là file ổ đĩa đã bị mã hóa của hệ điều hành Linux nên mình coppy vào máy ảo Linux để thực hiện.

```bash
$ sudo cryptsetup open --type luks b.luks test
```

Và dĩ nhiên vì là file đã mã hóa nên mình cần 1 password. File ảnh kèm theo cho chúng ta gợi ý về password và sau khi search tung google và nhìn thật kỹ vào bức anh, mình đã tìm ra đó là EFF passphrase. Để so khớp với các số đã cho mình dùng link này để có được password. Password là

`wages upturned flogging rinse landmass number`

Sau khi có được password mount ỗ đĩa và mình có được flag.

Flag: CTF{Look_Under_Keyboards_4_Secrets}

## thekey

Một file pcap bắt gói tin bàn phím nhập vào, một dạng khá phổ biến. Về tài liệu các bạn có thể tham khảo tại [đây](https://www.usb.org/sites/default/files/documents/hut1_12v2.pdf). Trong file pcap ở cột left over data sẽ cho chúng ta biết được mã hex của ký tự đươc nhập vào sau đó so sánh trong bảng ký tự của tài liệu mà mình để ở trên để decode ra được phím nào đã được nhập.

![IMG](/assets/img/blog/BSidesSF2019-3.PNG)

![IMG](/assets/img/blog/BSidesSF2019-4.PNG)

Tuy nhiên scappy có hỗ trợ cho nên mình dùng scappy. Source code:

```python
#!/usr/bin/python
from scapy.all import *
 
KEY_CODES = {
4:"A",
5:"B",
6:"C",
7:"D",
8:"E",
9:"F",
10:"G",
11:"H",
12:"I",
13:"J",
14:"K",
15:"L",
16:"M",
17:"N",
18:"O",
19:"P",
20:"Q",
21:"R",
22:"S",
23:"T",
24:"U",
25:"V",
26:"W",
27:"X",
28:"Y",
29:"Z",
30:"1",
31:"2",
32:"3",
33:"4",
34:"5",
35:"6",
36:"7",
37:"8",
38:"9",
39:"0",
40:"\n",
44:" ",
45:"_",
46:"=",
47:"{",
48:"}",
49:"|",
}
 
pkts = rdpcap("thekey.pcapng")
msg= ""
for packet in pkts:
    global msg
    hid_report = packet.load[-8:]
    key_code = ord(hid_report[2])
    ch = KEY_CODES.get(key_code, False)
    if ch:
        msg += ch
 
print msg
```

Sau khi decrypt mình nhận được một chuỗi khá lạ đó là:

> MC3AVIIM FLAAGTTXT
> ITTHE FLAAG IS CTFVBUUA\{\{MY_FAVORITTE_EDITOR_IS_VIM}HHHHHHHHHHHHHHHHHHHAUVI{UWQ`

Hm, mình stuck ở đoạn này vì có thể tác giả làm gì đó liên quan tới vim, tuy nhiên để ý lại thì tất cả đều clear nhưng chữ FAVORITTE lại sai chính tả nên mình sửa lại và submit thử sem sao. Ngạc nhiên là nó lại đúng và có 100 điểm.

Flag: CTF{MY_FAVOURITE_EDITOR_IS_VIM}

## bWF0cnlvc2hr YQ==

Đề bài cho mình file .eml nên mình dùng EML Reader trong Microsoft Store để mở nó. Trong file đính kèm mình có 2 file là hack.pgp và Matry_Oshka.key, bên cạnh đó đồng đội mình sử dụng notepad cũng có thấy một đoạn base64 và khi decrypt thì ra một ảnh qr code có nội dung __h4ck_the_plan3t__.

![IMG](/assets/img/blog/BSidesSF2019-5.PNG)

Sử dụng gpg trong linux để decrypt:

```bash
$ gpg --import Matry_Oshka.key

gpg: key 69D5F0038A289FB1: public key "Matry Oshka <Matry.Oshka@gmail.com>" imported
gpg: key 69D5F0038A289FB1: secret key imported
gpg: Total number processed: 1
gpg:               imported: 1
gpg:       secret keys read: 1
gpg:   secret keys imported: 1

$ gpg --list-key

pub   rsa2048 2019-03-01 [SC] [expires: 2021-02-28]
      0C7BCBBADF0C5273E797318069D5F0038A289FB1
uid           [ unknown] Matry Oshka <Matry.Oshka@gmail.com>
sub   rsa2048 2019-03-01 [E] [expires: 2021-02-28]

$ gpg -o out --decrypt hack.pgp 
gpg: encrypted with 2048-bit RSA key, ID E5CAE1F0A16B0E7A, created 2019-03-01
      "Matry Oshka <Matry.Oshka@gmail.com>"
```

Check thử xem thì file out là file zip

```bash
$ file out
out: Zip archive data, at least v2.0 to extract
```

Sau khi extract file zip và dùng binwalk, mình thấy có một file lzip ở offset thứ 10, tiến hành dump file và giải nén mình có được một file pdf chứa ảnh nền windows xp huyền thoại. Sau khi dùng adobe acrobat, mình extract được file ảnh ra ngoài. Đoạn này khá là tiếc khi trong cuộc thi mình chỉ extract file jpg nên stuck và không thể giải quyết được lúc giải diễn ra. Chỉ hôm sau mình extract bằng file bitmap thì mới có thể làm đúng hướng khi admin nói mình nên tra wiki ảnh gốc. Dùng stegsolve mình có được 1 mã qr lỗi.

![IMG](/assets/img/blog/BSidesSF2019-6.PNG)

Sau khi sửa lại bằng cách đổi pixel trắng thành đen và ngược lại, mình có được một mã qr hoàn hảo.

![IMG](/assets/img/blog/BSidesSF2019-7.PNG)

Và mình có được đoạn base64:


> /Td6WFoAAATm1rRGAgAhARwAAAAQz1jM4ELCAORdABhgwwZfNTLh1bKR4pwkkcJw0DSEZd2BcWATAJkrMgnKT8nBgYQaCPtrzORiOeUVq7DDoe9feCLt9PG-MT9ZCLwmtpdfvW0n17pie8v0h7RS4dO/yb7JHn7sFqYYnDWZere/6BI3AiyraCtQ6qZmYZnHemfLVXmCXHan5fN6IiJL7uJdoJBZC3Rb1hiH1MdlFQ/1uOwaoglBdswAGo99HbOhsSFS5gGqo6WQ2dzK3E7NcYP2YIQxS9BGibr4Qulc6e5CaCHAZ4pAhfLVTYoN5R7l/cWvU3mLOSPUkELK6StPUBd0AABBU17Cf970JQABgALDhQEApzo4PbHEZ/sCAAAAAARZWg==

Decode mình có được một file 7zip và bên trong là một file text chứa mã nhị phân. Decode mã nhị phân theo các bước:
- Nhị phân -> ascii, mình có được dãy số
- Bát phân -> thập phân
- Thập phân -> ascii có đươc mã hex
- Từ hex -> base 64
- Tử base 64 -> base 85
- Từ base 85 -> flag

![IMG](/assets/img/blog/BSidesSF2019-8.PNG)

## Zippy

File pcap cho mình biết được câu lệnh unzip file flag.zip. Và file flag.zip được lưu từ gói tin thứ 6 dến gói tin thứ 13. Extract file zip và giải nén với password: supercomplexpassword, mình có được flag.

Flag: CTF{this_flag_is_your_flag}
