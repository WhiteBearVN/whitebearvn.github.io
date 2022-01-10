---
title : "USB PCAP - Phần 1 Bàn phím"
author : "F4c3l3ss_"
date : 2021-04-07
tags : ["research"]
categories: [Research, Forensics]
---

Một bài chia sẻ, tổng hợp kiến thức cá nhân giúp mình hoàn thành các thử thách truyền thông tin sử dụng giao thức USB sang PC/Laptop được capture thông qua Wireshark hoặc T-shark và lưu dưới tệp tin Pcap/Pcapng.

## 0x01: Các định dạng truyền dữ liệu

![IMG](/assets/img/blog/1_DangUSB.JPG)

Có 4 định dạng truyền dữ liệu thông qua USB đó là:

    - Control Transfers (0x02)
    - Interrupt Transfers (0x01)
    - Isochronous Transfers (0x00)
    - Bulk Transfers (0x03)

Với các mã hex tương ứng cho câu lệnh lọc gói tin trên Wireshark, ví dụ `usb.transfer_type == 0x01` sẽ giúp chúng ta lọc ra toàn bộ các gói tin được gửi đi dưới phương thức Interrupt.

![IMG](/assets/img/blog/2_VdInterrupt.JPG)

### Control Transfers

Đây là dạng giao thức được sử dụng cho việc thiết lập USB và trạng thái của thiết bị cũng như endpoint.

### Interrupt Transfers

Dạng giao thức được dùng để truyền dữ liệu từ bàn phím, chuột đến máy tính

### Isochronous Transfers

Dạng giao thức được dùng để truyền tải âm thanh đến các thiết bị ngoại vi âm thanh như tai nghe, loa,...

### Bulk Transfers

Dạng giao thức được dùng để truyền tải dữ liệu lớn như là một bức ảnh được gửi từ máy tính qua máy in hoặc từ máy quét sang máy tính.

## 0x02: Đọc dữ liệu được nhập từ bàn phím

### Xác định và cách đọc dữ liệu

Vì dữ liệu truyền vào từ bàn phím thuộc `interrupt transfers` nên để xác định được các gói tin liên quan mình dùng lệnh `usb.transfer_type == 0x01` trong Wireshark để filter gói tin. Vì bàn phím là thiết bị có tốc độ thấp nên dữ liệu sẽ luôn ở dạng 8 bytes xx:xx:xx:xx:xx:xx:xx:xx.

![IMG](/assets/img/blog/3_dinhdangbytes.JPG)

Thông thường thông tin của thiết bị sẽ được ghi trong các gói tin `GET DESCRIPTOR Request/Response DEVICE` tuy nhiên những gói tin này nằm ngoài phạm vi của bài viết nên mình sẽ không đề cập đến mà chỉ tập trung vào cách lấy dữ liệu từ pcap/pcapng.

Trong hình trên ta thấy ở gói tin số 1 có thông tin `00:00:09:00:00:00:00:00`, với `09` tương ứng với chữ `f` trên bàn phím. Tại sao lại như vậy? Đó là do thống nhất giữa máy tính và bàn phím, thông tin về mã hex tương ứng với từng bàn phím, các bạn có thể tham khảo tại đây: [https://usb.org/sites/default/files/documents/hut1_12v2.pdf](https://usb.org/sites/default/files/documents/hut1_12v2.pdf) trang 53.

Tuy nhiên chú ý thì gói tin số 10, lại có `02` đứng đầu, vậy nó khác gì so với gói tin 01 có bytes đầu là `00`? 

![IMG](/assets/img/blog/3_dinhdangbytes2.JPG)

Đó là khi người dùng đè `shift` + `something`. Trong trường hợp trên `2F` tương ứng với dấu `[` nhưng vì đè shift đằng trước nên kí tự đúng sẽ là `{`.

Vậy tất cả các bàn phím đều giống nhau? Câu trả lời là `không`. Mỗi quốc gia sử dụng những ngôn ngữ, kiểu chữ khác nhau sẽ có những mã hex tương ứng với từng kí tự của ngôn ngữ đó. Ví dụ như crush 7 năm của mình là người rất thích, và là chuyên gia tiếng Nhật nên mình tìm hiểu thì ra được bảng này: [http://hp.vector.co.jp/authors/VA003720/lpproj/others/kbdjpn.htm](http://hp.vector.co.jp/authors/VA003720/lpproj/others/kbdjpn.htm). Đây là trang web so sánh sự khác, giống nhau của bàn phím tiếng Nhật với bàn phím theo chuẩn quốc tế mà chúng ta hay thường sử dụng.

Có bạn sẽ thắc mắc vậy bàn phím của Việt Nam thì sao? Mình xin trả lời việc chúng ta đánh máy bằng Telex hay VNI để ghi tiếng Việt có dấu là xử lý ở mức độ application, không liên quan đến việc input data từ bàn phím. Dù bạn gõ tiếng Việt nhưng nếu thử capture bằng Wireshark thì chắc chắn gói tin bạn nhận được vẫn sẽ theo dạng kí tự nhập vào. Ví dụ mình gõ VNI `chào` thì khi capture và xuất dữ liệu, kết quả chúng ta nhận được là `chao2`.

8 bytes dữ liệu này sẽ luôn xuất hiện dưới 1 trong 2 trường là: `Leftover Capture data` hoặc `HID Data` tại thông tin của mỗi gói tin; sự khác biệt này hiện tại mình chưa tìm ra nguyên nhân, nên khi xem xét một gói tin mình luôn làm theo flow và theo dõi data trong đó để xác định.

### Xuất kết quả 

Chúng ta không thể cứ đọc từng line và so sánh từng dòng code với từng mã hex trong bảng được. Vì vậy cách nhanh nhất là xuất bằng command line. Mình thường sử dụng `tshark` để xuất thông tin sau đó decode.

Câu lệnh `tshark`:

```bash
$ tshark -r ./usb.pcap -Y 'usb.capdata && usb.data_len == 8' -T fields -e usb.capdata | sed 's/../:&/g2'
```

Ở một số phiên bản sẽ in sẵn dấu `:` cho từng bytes thì các bạn có thể xóa lệnh pipe sed.

Tiếp theo, mình decode những mã hex vừa được output.

EzPz.

Một số thử thách CTF liên quan đến vấn đề này các bạn có thể thử là:

    - PicoCTF 2017_Just Keyp Trying
    
    - IceCTF2016
    
    - Bsidesf CTF_The Key.

Cheer (☞ﾟヮﾟ)☞ ☜(ﾟヮﾟ☜)
