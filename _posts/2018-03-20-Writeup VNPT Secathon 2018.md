---
title: "[Writeup] VPNT Secathon 2018 - Misc"
author: F4c3l3ss_
date: 2018-03-20
categories: [Writeup,VNPTSecathon2018]
tags: [writeup,forensics,ctf]
math: true
mermaid: true
---

## Chayngaydi

Trong bài này ban tổ chức cho chúng ta đoạn teaser trailer Chạy ngay đi của Sếp.

![IMG](/assets/img/blog/hinh1.jpg)



Ngay khi nhìn vào dung lượng của tập tin mp4, thì mình biết chắc chắn rằng nó đang giấu một file gì đó.

Mở file mp4 bằng binwalk hay foremost cũng không thấy gì khác biệt, nên mình đành làm nước cuối cùng đó là tải luôn đoạn teaser của Sếp về để xor, và ngay lập tức mình có được file wav đã bị đổi header. Và nó bị đổi header thì chúng ta phải sửa lại cho đúng

từ

![IMG](/assets/img/blog/hinh2.PNG)

thành

![IMG](/assets/img/blog/hinh3.PNG)

sau khi có được file wav, trong lúc thi mình có nghĩ đến kĩ thuật lsb và đã sử dụng wavesteg để lấy flag nhưng đáng buồn là mình chỉ sử dụng 1 lsb để decode và ra được 1 nữa flag.

![IMG](/assets/img/blog/hinh4.PNG)

 Cho đến lúc chiều khi ban tổ chức có hint là ack mình vẫn không biết đó là kĩ thuật gì. Sau khi cuộc thi kết thúc mình có hỏi và được mấy anh trong Zepto team thì mới biết đó là kĩ thuật dubstep data encoding, một kĩ thuật rất mới ( mình từng có lướt qua twitter của DEFCON và thấy kĩ thuật này nhưng hình như lúc đó đang cày quest book TI8 nên bỏ qua luôn thì phải 😭 ). Các bạn có thể tham khảo kĩ thuật này tại [đây](https://blog.benjojo.co.uk/post/encoding-data-into-dubstep-drops).



Sau khi follow theo bài viết của tác giả mình decode ra được đoạn hex:

#####53656341743b0a0d3a230d03021f2b013a42091c310836065c2d012a121a1106154c0

vẫn chưa phải là flag 😰.

Ngẫm một hồi mình lại nhìn đến nửa flag mình đã tìm được trong lúc thi tìm thêm nhưng mảnh ghép ??? Hmm có khi nào là dùng nhiều lsb hơn? Mình đã thử dùng 2 lsb và cái kết:

![IMG](/assets/img/blog/hinh5.PNG)

Mình ngớ người khi ra kết quả như vậy, thật sự là mình rất cay, bởi vì trong lúc thi mình quá mất bình tĩnh khi chỉ tìm được 1 nữa flag để rồi rối trí trong việc tìm 1 nữa còn lại. 😐

![IMG](/assets/img/blog/hinh6.PNG)

Cập nhật 29/05/2018: Sau khi mình viết bài này thì được tác giả của bài góp ý rằng flag của lsb chưa đúng định dạng và flag là xor(lsb,ack) nên mình đã làm lại và xor 2 đoạn trên thì mình đã ra được flag chính xác:

A huge thank to the changllenge's author 👍

Thông tin quan trọng: Trong tầm tháng 6 hoặc 7 đội mình sẽ tổ chức 1 giải CTF với mục đích giao lưu  rất mong nhận được sự tham gia của các bạn. 👍

## Decrypted file

"Bài này mình viết lại theo solution của anh Nhật Hoàng, một thành viên khác trong đội. Mình chỉ đóng góp phần resize file bmp để ra được flag. "

Đề bài chúng ta có 1 file exe dùng để mã hoá và 1 file bị mã hoá. Dùng IDA để xem nó hoạt động như thế nào



Mình ko biết hàm sub_4010A0 làm gì nên bỏ qua việc phân tích. Giờ đến hàm chính của bài đó là hàm sub_4017D0. 


Ta thấy rằng bài này chủ yếu là phép xor giữa file mình nhập vào và file key. Và nếu gặp byte null thì giữ nguyên không xor.

Mình thấy ý tưởng bài này khá giống bài ransomware trên reversing.kr nên mình đoán ràng file bị mã hóa là file PE. Trong file PE thì có 1 đoạn là mặc định luôn luôn có:


Nên mình lấy đoạn này trong file PE khác xor vs file bị mã hóa => key 
Mình thấy key là 1 đoạn hex =>  
Thấy rằng đoạn hex được lặp lại => cái này là key = 
#####5c7ab41e37efc4e816ebfcbc70071aa2
Dùng key để giải file bị mã hóa
Mình làm đến đoạn này tưởng rằng đã xong nhưng mà file PE sau khi giải chạy ko được.
Ngồi cả buổi để fix lỗi cho file đó chạy nhưng vẫn không thành công. 
Về làm lại và được hint từ ban tổ chức thì file đó chạy ko được nhưng sử dụng CFF explorer thì có thấy 1 file bitmaps



Và để đọc được file bmp này thì cần phải resize lại cho đúng với độ phân giải. Mình dùng 010 editor và sau nhiều lần thử thì tăng chiều rộng lên 2 lần và giảm chiều cao 2 lần sẽ ra được flag:



Code tìm key và decrypt file, các bạn tham khảo: 
```python
f = open('dec.exe','rb').read()
g = open('file','rb').read()
k = open('flag.exe','wb')

#find key
data = f[78:78+40]
print data
data1 = g[78:78+40]
s = ''
for i in range(len(data)):
    s += chr(ord(data[i])^ord(data1[i]))
print s 
# key = 32
# =>bat dau cua key la vi tri thu 96 ma ta lay data tu vi tri 78 => 78+18 la vi tri bat dau cua key

# decrypt file
s = ''
key = '5c7ab41e37efc4e816ebfcbc70071aa2'
for i in range(len(g)):
    if ord(g[i])!= 0:
        s += chr(ord(g[i])^ord(key[i%len(key)]))
    else:
        s += g[i]

k.write(s)
k.close()

print 'solve !!!!!!'
```
