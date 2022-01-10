---
title : "[Writeup] Diễn tập ATTT Ngân hàng Nhà Nước 31/03/2021"
author : "F4c3l3ss_"
date : 2021-03-31
tags : ["writeup"]
categories: [Writeup]
---

## 0x01: Ứng cứu sự cố, điều tra số

### 1. Time2Attack

Trong bài này, ban tổ chức (BTC) cung cấp 1 file pcap. Sau khi mở bằng wireshark ta có 1 file zip với tên `time.zip`. Nhận thấy đây là 1 file zip yêu cầu password để mở đồng thời thông tin trước đó BTC yêu cầu tool bruce force nên mình sử dụng `john the ripper` để dò password xem sao.

```bash
$ zip2john time.zip > time.hash
$ john  --wordlist=rockyou.txt -format:zip time.hash
```

Và mình có được password là `!@#$%`. Tiếp đến, trong file zip là một file text với nội dung gần như là mã base64, nhìn lại file text có tên là rot-me, vậy có thể là sự xáo trộn của 2 mã này, dùng `CyberChef` decrypt theo flow: data -> rot13 -> base64 -> image, mình get được ảnh có chứa flag

![IMG](/assets/img/blog/1_dientapnhnn_31032021.JPG)

![IMG](/assets/img/blog/flag_decode.jpg)

### 2. Meow 

Đề bài yêu cầu tìm thời gian mà user meow được tạo ra theo log event của Windows. Ban đầu mình sử dụng tìm kiếm theo event ID tạo user là 4720 nhưng không tìm thấy event nào, cũng lúc ấy một người anh trong team nói log AD server, ngẫm lại thì 4720 là log của máy local, nếu dùng AD thì sẽ không thể có log 4720 được nên mình dùng chức năng tìm kiếm mình có tìm thấy 2 event tạo user thông qua 2 tiến trình `net.exe` và `net1.exe`. Search google thì 2 tiến trình này là gần giống nhau trong đó lệnh `net.exe` đã được sửa để vá lỗi Y2K (sự cố máy tính năm 2000) và `net1.exe` là phiên bản cũ hơn để sử dụng tính tương thích ngược. Và cũng không xác định được time nào chính xác hơn mà BTC cho 10 lần submit nên mình submit cả 2 và đáp án là time đã call `net1.exe`

Flag: ATTT{24/03/2021-09:28:18}

### 3. Who let the cats in?

Sử dụng đáp án của câu trên, mình có được event đã tạo user từ đó lấy được ```Logon ID:	0x175CBA```, trace back Logon ID này với time kết nối gần nhất.

![IMG](/assets/img/blog/2_dientapnhnn_31032021.JPG)

Flag: ATTT{192.168.95.133-50921}

### 5. Command Sharing Point

Ngay khi nhận được file pcap, nhận thấy trong các gói tin có chứa shellcode, nên mình export toàn bộ file cũng như xem số thứ tự packet. Đầu tiên có IP: `10.3.1.65` truy cập và thực hiện command trên server và exploit 2 user là `hro1` cùng `sp_farm`.

![IMG](/assets/img/blog/3_dientapnhnn_31032021.JPG)

![IMG](/assets/img/blog/4_dientapnhnn_31032021.JPG)

Nhưng khi nhìn lại tại gói streams 749 đã cho mình thấy rằng đã tồn tại file `reverse_shell.txt`. Vậy có thể trên máy đã bị exploit từ trước đó và IP `10.3.1.65` là địa chỉ C2. Sau một thời gian tìm kiếm đồng đội mình có tìm thấy tại streams 340 có lệnh tạo user và stream 1233 có tải shell từ IP `10.3.1.65`.

![IMG](/assets/img/blog/5_dientapnhnn_31032021.JPG)

![IMG](/assets/img/blog/6_dientapnhnn_31032021.JPG)

Ngoài ra khi nhìn thấy payload, google search ra được [CVE-2020-1181](https://www.thezdi.com/blog/2020/6/16/cve-2020-1181-sharepoint-remote-code-execution-through-web-parts). Và các session để exploit sử dụng login là user `hr01`, việc còn lại là xác định IP, ban đầu team mình nghĩ là IP `192.168.95.130` vì đây là IP thực hiện exploit, tuy nhiên theo đáp án của BTC thì IP đúng cho flag là IP C2: `10.3.1.65`.

Flag: ATTT{CVE-2020-1181/hr01/10.3.1.65}

### 6. Am I qualified to be your hacker?

Bài này BTC cung cấp file save của Process Monitor, quan sát Process Tree, ta thấy tiến trình EQNEDT32.EXE đang call tiến trình con là powershell và cmd, nhìn phát ra ngay là CVE-2017-11882. 

![IMG](/assets/img/blog/7_dientapnhnn_31032021.JPG)

Phần còn lại là xác định file Word mà user hr01 đã mở, ta để ý time của tiến trình EQNEDT32.EXE start lúc 07:14:03, trong khoảng thời gian này chỉ có file CV-HuyTQ.doc là hoàn toàn phù hợp cho điều kiện trên.

![IMG](/assets/img/blog/8_dientapnhnn_31032021.JPG)

Flag: ATTT{CV-HuyTQ.doc;CVE-2017-11882}



## 0x02: Pentest

Vì server đã bị BTC đóng nên mình không viết chi tiết được nên chỉ nêu flow và cách thực hiện.

### 7. Magic flag in code

Bài này exploit thông qua lỗi PHP Deserialization - Phar Files. 

Ban đầu, tạo một shell code và lưu dưới dạng .jpg vì code chỉ check extenstion mà không check thông qua magic bytes. Tìm đến khai khác PHP Deserialization - Phar Files để từ file .jpg -> file .php và đọc flag tại md5.php.

Chi tiết các bạn có thể coi video của IppSec: [https://www.youtube.com/watch?v=fHZKSCMWqF4](https://www.youtube.com/watch?v=fHZKSCMWqF4)

### 10. Hide and Seek

Bài này sau khi sử dụng dirsearch, mình nhận thấy nếu muốn truy cập admin đều sẽ bị 403, tuy nhiên tại response trả về của server, mình nhận ra server đang chạy ApacheWebser tại port 8000 và dùng Nginx làm Reverse proxy. Loay hoay cách bypass theo dạng HttpSmugling thì đồng đội mình có dùng hint và thu về h2c. Kết hợp lại google mình thu về được: [https://github.com/BishopFox/h2csmuggler](https://github.com/BishopFox/h2csmuggler)

```bash
./h2csmuggler.py -x https://link.challenge http://localhost/admin
```

Cảm ơn BTC!

Cheer (☞ﾟヮﾟ)☞ ☜(ﾟヮﾟ☜)
