---
title : "[Writeup] Mates SS2 Round 4 - Forensics caigidzay"
author : F4c3l3ss_
date : 2018-05-01
tags: [writeup,forensics,ctf]
categories: [Writeup]
math: true
mermaid: true
---
Trong thử thách này để bài là một file dump, mbr/dos nên mình sẽ dùng auto spy để mở nó xem sao. Đập vào mắt mình là flag.jpg.

![IMG](/assets/img/blog/hinh01.jpg)

Mình tốn kha khá thời gian với bức ảnh này vì lúc đầu tưởng đây là thử thách về stegano tuy nhiên sau khi đọc lại miêu tả đề và thiết nghĩ thì nếu stegano thì ban tổ chức chỉ cần cho file ảnh là xong chứ ai lại cú lừa cho file 6gb để làm stegano :D

Đọc lại miêu tả thì Mr.John gửi file flag hay đại loại cái gì đó qua mail thì sẽ hợp lý hơn.
Sau một hồi tìm banh mắt giữa 857 cái mail, giỡn chứ mình bấm chữ j để kiếm john trước  chứ sao đọc hết nhiêu đó cái mail được :D, thì mình thấy được rằng mail này được gửi cho leia, kiểm tra xem có mail của leia không nào.

![IMG](/assets/img/blog/hinh02.PNG)

Ngay lập tức ta biết đây là người john đã gửi file x gì gì đó, extract hết ra và mở file Outbox.dbx bằng Turgs DBX Viewer ta càng khẳng định đây là file mà John đã leak ra ngoài.

![IMG](/assets/img/blog/hinh03.PNG)

Trong phần attachments có 1 file doc tên là org_contract.doc. Dùng auto spy tìm theo tên file và extract ra. Oh có password lúc đó cứ nghĩ tới cái hình flag đầu tiên kèm theo cái game trong nội dung mail trên mình thử các kiểu dota2 nhưng không đúng lúc đó kiểu ơ btc xài chiêu à. Nhưng khi đọc lại mail thì hình như mình đã bỏ qua dòng

About the password to open the file, you should overlook somewhere, it's already included in the file

thử tìm lại trong file doAbout the password to open the file, you should overlook somewhere, it's already included in the file, thử dùng lệnh strings trong linux ez ta có 1 link imgur:

https://imgur.com/a/qHeqyqm

Lại một lần nữa Doãn huynh xuất hiện, cú lừa lần 2 chăng? :S Thử tải về và dùng zsteg, wew lần này là thật :v đơn giản ta có được password của file doc:

![IMG](/assets/img/blog/hinh4.PNG)

```Would yout like to open the gate to the super secret treasure?
You must be a chosen one, trust me, otherwise you're not qualified enough. Poor you!
unless you answered this question, the ancient guardian might take a look at it. 
It's a key to heal up your life.
"What is the largest value that a signed int32 can store?"
It may starts with the number two, I mean the decimal.
```

Đó là số int32 lớn nhất: 2147483647

Cứ ngỡ có được flag để submit nào ngờ lại phải thêm 1 bước nữa là decrypt cái đống webdings trong file doc @@ :

![IMG](/assets/img/blog/hinh05.PNG)

Mình đổi sang wingdings rồi dùng trang này để dịch ra flag

![IMG](/assets/img/blog/hinh6.JPG)

Lúc ra flag mình đang học thực hành trên trường nên để lap 1 bên để dùng máy trường chọn mấy cái hình cho tiện nên chụp đại bằng điện thoại, mọi người thông cảm :))

