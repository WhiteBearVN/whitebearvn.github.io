---
title : "[Lỗi vu vơ] Thay đổi số tiền chuyển khoản SMS gửi về của ngân hàng MQĐ"
author : "F4c3l3ss_"
date : 2021-05-16
tags : ["research, android"]
categories: [Research, Mobile, Android]
---

Vài hôm trước mình có được một người anh nhờ reverse và bypass các tính năng bảo mật cho ứng dụng của ngân hàng MQĐ trên Android; cũng lâu rồi chưa làm gì đụng lại đến Android nên mình thấy đây là một cách tốt để research thêm về các kĩ thuật trên Android.

## 0x01: Bypass 

Về cơ bản mình thấy app được bảo mật và kiểm tra môi trường mà ứng dụng dược chạy rất kỹ, thông qua một nhà cung cấp thư ba mà ngân hàng MQĐ đã tích hợp vào source code chỉ cần vi phạm thì app sẽ dừng chạy hoặc crash. Các trường mà ứng dụng thứ ba kiểm tra bao gồm:

    - Ứng dụng phải đang chạy trên thiết bị thật (Anti-emulator)
    - Thiết bị có bị root hay chưa (Anti root)
    - Ứng dụng có bị chặn gói tin hay không (Cert pinning)

Cách thức bypass mình không tiện nói ở bài đăng này; có thể sẽ là một bài đăng trong tương lai hay cũng có thể là không có vì nhiều nguyên nhân.

## 0x02: Nghịch ngợm

Sau khi bypass toàn bộ mình đi vào trong và thử intercept các tính năng quan trọng; ứng dụng ngân hàng mà phải thử ngay giao thức chuyển tiền đầu tiên chứ 😌
Khi check sơ qua thì mình thấy luồng thực thi khi gọi API chuyển tiền của ứng dụng là như sau:

    - Transaction -> CheckPayment1 -> GenSMS_OTP -> CheckPayment2+OTP -> CheckBalance -> DoTransaction

Tuy nhiên trong quá trình thì mình nhận thấy việc GenSMS_OTP hoàn toàn không kiểm tra bước CheckPayment1 và không có các hàm mã hóa khi xử lý ở client để đảm bảo tính toàn vẹn nên có thể dễ dàng thay đổi các tham số.

![IMG](/assets/img/blog/POC.PNG)

Tràn số 0 luôn mà, híc ước gì đống trên là của mình chuyển thật. Một thanh niên khố rách áo ôm cho hay. 😥😪

Lỗi này mình đã nhờ người anh của mình report lại với phía bên kia nhưng không nhận được phản hồi và cũng đã hơn 14 ngày tính từ ngày report nên mình up lên đây cho mọi người cùng đọc vì lỗi này tính ra mức impact cũng không cao lắm chủ yếu có thể dùng là đi phishing hay lừa đảo bằng ảnh chụp màn hình thôi. 

Cheer (☞ﾟヮﾟ)☞☜(ﾟヮﾟ☜)
