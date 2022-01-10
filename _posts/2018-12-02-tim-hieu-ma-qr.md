---
title: QR Code
author: F4c3l3ss_
date: 2018-12-02
categories: [Research, Forensics]
tags: [qrcode,forensics]
math: true
mermaid: true
---

**QR** là viết tắt của quick response, là một loại mã vạch đặc biệt có thể mã hóa thông tin như số, chữ cái, và kí tự kanji được tạo bởi công ty Denso-Wave là một công ty con của tập đoàn Toyota.

Hiện nay có rất nhiều loại QR code như:
- QR code model 1 & 2: loại chúng ta thường thấy.
- Micro QR code: một loại QR code nhỏ.
- SQRC, iQR code, frame QR code,...

Trong bài viết này mình sẽ chỉ tập trung vào QR code model 2 vì từ tháng 9 năm 2006 ISO ISO/IEC 18004 cũng đã loại việc đọc mã QR code model 1 (đoạn này mình không biết nói sao cho đúng ý nhất nhưng vì sự vượt trội của QR model 2 nên phải loại model 1 để có tính thống nhất ấy).
# QR model 1
Mã QR code gốc có thể mã hóa 1167 số và có tối đa 14 phiên bản (73x73)

# QR model 2
Bản cải thiện của model 1 có thể mã hóa 7089 số và có tối đa 40 phiên bản (177x177).

Mã QR luôn có hình vuông và được phân chia làm 40 phiên bản dựa vào kích thước của chúng.

- Phiên bản 1 sẽ có kích thước là 21x21.

- Phiên bản 2 sẽ có kích thước là 25x25.

- Phiên bản 3 sẽ có kích thước là 29x29.

Và cứ mỗi phiên bản sau sẽ lớn hơn phiên bản trước 4 đơn vị và đến phiên bản tối đa là 40 với kích thước là 177x177.

Kích thước của mã QR được xác định bởi công thức:

>	**(V - 1) * 4 + 21**

_với V là phiên bản mã QR_

Vây với phiên bản 30 ta sẽ có kích thước là 137x137.

## Về kích thước:
Chắc hẳn các bạn sẽ thắc mắc tại sao có những ảnh phiên bản khác nhau nhưng lại cùng kích thước. Lý do là kích thước đó phụ thuộc vào bạn quy định bao nhiêu pixel cho một hàng, cột. Nếu bạn quy định 1 pixel là 1 hàng và 1 cột thì kích thước của 2 ảnh chắc chắn sẽ khác nhau. Vì vậy thay vì quyết định dựa trên số pixel của bức ảnh hãy dựa vào số hàng và cột của QR code đó. Trong bài viết này mình dựa vào hàng và cột như vậy sẽ dễ phân biệt và thuận tiện hơn trong bài viết.

## Về dung lượng:
Dung lượng của mã QR phụ thuộc vào cấp độ sửa lỗi và phiên bản. Có bốn chế độ dữ liệu bao gồm: số, chữ và số, nhị phân, chữ kanji. Và bốn chế độ sửa lỗi: L, M, Q, H (mình sẽ nói kỹ hơn ở dưới). Các bạn có thể truy cập trang này đễ xem dung lượng tối đa mà mã QR có thể chứa dựa trên cấp sửa lỗi.

## Về màu sắc mã:
QR luôn có hai màu là đen và trắng trong đó đen sẽ đại diện cho bit 1 còn trắng đại diện cho bit 0.

Cấu trúc của mã QR được thể hiện qua hình sau:

![cautruc](/assets/img/blog/1_cautruc.png)


## Vùng finder

Vùng finder là 3 ô vuông đen lớn ở ba góc trái trên, phải trên và trái dưới  bất kể phiên bản mã QR nào. Vùng này giúp cho thiết bị đọc mã xác định đươc ảnh QR và thực hiện việc đọc dữ liệu. Chính nhờ có vùng này mà khi chúng ta quay ảnh QR ngang, chéo thì điên thoại chúng ta vẫn có thể đọc được.
Mỗi ô vuông của vùng finder luôn có viền màu đen bên ngoài kích thước là 7 bên trong là một hình vuông trắng kích thước 5 và ở giữa là ô vuông đen 3x3.

![finder](/assets/img/blog/2_finder.png)

Để xác định ví trị đặt 3 ô finder ta có công thức tính như sau:

- Góc trái trên luôn luôn đặt tại vị trí (0,0)

- Góc trái dưới đặt tại ví trí (x,0)

- Góc phải trên đăt tại ví trí(0,y)

Trong đó x,y được tính bằng công thức:

>	**(V - 1) * 4 + 14**

_với V là phiên bản mã QR_

![finder_example](/assets/img/blog/3_finder_example_1.png)

Phiên bản 1

## Vùng separators
Là vùng trắng bao quanh vùng finder giúp tách vùng finder và phần còn lại của mã QR.

![separators](/assets/img/blog/4_separators.png)

## Vùng alignment
Có công dụng tương tự như vùng finder nhưng nhỏ hơn. Vùng này chỉ xuất hiện từ phiên bản 2 đến phiên bản 40. Vùng alignment có cấu trúc gần giống với finder, là một ô vuông có viền màu đen cạnh là 5 bên trong là ô vuông trắng cạnh là 3 và ở giữa là ô vuông 1x1.
Về vị trí vùng alignment có vị trí cố định và khác vùng finder. Tuy nhiên khi đặt alignment cần chú ý nó không được trùng với các vị trí cấu trúc khác. Như ví dụ dưới đây vùng alignment của phiên bản 2 có thể đặt tại tọa độ là sự kết hợp của 2 số 6 và 18. Nhu vậy ta có 4 vị trí (6,6), (6,18), (18, 6), (18, 18) tuy nhiên chỉ có vị trí (18, 18) là có thể đặt vì ba vùng kia trùng với ví trị của vùng finder.

![alignment-exclusion](/assets/img/blog/5_alignment-exclusion.png)

![alignment-correct](/assets/img/blog/6_alignment-correct.png)



Một ví dụ khác về phiên bản 8:

![alignment-example](/assets/img/blog/7_alignment-example.png)

Bảng các số kết hợp của vùng alignment


```
+-----------+------------------------------------+
| Phiên bản |      Chỉ số hàng để tạo ra cột     |
+-----------+---+----+----+----+-----+-----+-----+
| 2         | 6 | 18 |    |    |     |     |     |
+-----------+---+----+----+----+-----+-----+-----+
| 3         | 6 | 22 |    |    |     |     |     |
+-----------+---+----+----+----+-----+-----+-----+
| 4         | 6 | 26 |    |    |     |     |     |
+-----------+---+----+----+----+-----+-----+-----+
| 5         | 6 | 30 |    |    |     |     |     |
+-----------+---+----+----+----+-----+-----+-----+
| 6         | 6 | 34 |    |    |     |     |     |
+-----------+---+----+----+----+-----+-----+-----+
| 7         | 6 | 22 | 38 |    |     |     |     |
+-----------+---+----+----+----+-----+-----+-----+
| 8         | 6 | 24 | 42 |    |     |     |     |
+-----------+---+----+----+----+-----+-----+-----+
| 9         | 6 | 26 | 46 |    |     |     |     |
+-----------+---+----+----+----+-----+-----+-----+
| 10        | 6 | 28 | 50 |    |     |     |     |
+-----------+---+----+----+----+-----+-----+-----+
| 11        | 6 | 30 | 54 |    |     |     |     |
+-----------+---+----+----+----+-----+-----+-----+
| 12        | 6 | 32 | 58 |    |     |     |     |
+-----------+---+----+----+----+-----+-----+-----+
| 13        | 6 | 34 | 62 |    |     |     |     |
+-----------+---+----+----+----+-----+-----+-----+
| 14        | 6 | 26 | 46 | 66 |     |     |     |
+-----------+---+----+----+----+-----+-----+-----+
| 15        | 6 | 26 | 48 | 70 |     |     |     |
+-----------+---+----+----+----+-----+-----+-----+
| 16        | 6 | 26 | 50 | 74 |     |     |     |
+-----------+---+----+----+----+-----+-----+-----+
| 17        | 6 | 30 | 54 | 78 |     |     |     |
+-----------+---+----+----+----+-----+-----+-----+
| 18        | 6 | 30 | 56 | 82 |     |     |     |
+-----------+---+----+----+----+-----+-----+-----+
| 19        | 6 | 30 | 58 | 86 |     |     |     |
+-----------+---+----+----+----+-----+-----+-----+
| 20        | 6 | 34 | 62 | 90 |     |     |     |
+-----------+---+----+----+----+-----+-----+-----+
| 21        | 6 | 28 | 50 | 72 | 94  |     |     |
+-----------+---+----+----+----+-----+-----+-----+
| 22        | 6 | 26 | 50 | 74 | 98  |     |     |
+-----------+---+----+----+----+-----+-----+-----+
| 23        | 6 | 30 | 54 | 78 | 102 |     |     |
+-----------+---+----+----+----+-----+-----+-----+
| 24        | 6 | 28 | 54 | 80 | 106 |     |     |
+-----------+---+----+----+----+-----+-----+-----+
| 25        | 6 | 32 | 58 | 84 | 110 |     |     |
+-----------+---+----+----+----+-----+-----+-----+
| 26        | 6 | 30 | 58 | 86 | 114 |     |     |
+-----------+---+----+----+----+-----+-----+-----+
| 27        | 6 | 34 | 62 | 90 | 118 |     |     |
+-----------+---+----+----+----+-----+-----+-----+
| 28        | 6 | 26 | 50 | 74 | 98  | 122 |     |
+-----------+---+----+----+----+-----+-----+-----+
| 29        | 6 | 30 | 54 | 78 | 102 | 126 |     |
+-----------+---+----+----+----+-----+-----+-----+
| 30        | 6 | 26 | 52 | 78 | 104 | 130 |     |
+-----------+---+----+----+----+-----+-----+-----+
| 31        | 6 | 30 | 56 | 82 | 108 | 134 |     |
+-----------+---+----+----+----+-----+-----+-----+
| 32        | 6 | 34 | 60 | 86 | 112 | 138 |     |
+-----------+---+----+----+----+-----+-----+-----+
| 33        | 6 | 30 | 58 | 86 | 114 | 142 |     |
+-----------+---+----+----+----+-----+-----+-----+
| 34        | 6 | 34 | 62 | 90 | 118 | 146 |     |
+-----------+---+----+----+----+-----+-----+-----+
| 35        | 6 | 30 | 54 | 78 | 102 | 126 | 150 |
+-----------+---+----+----+----+-----+-----+-----+
| 36        | 6 | 24 | 50 | 76 | 102 | 128 | 154 |
+-----------+---+----+----+----+-----+-----+-----+
| 37        | 6 | 28 | 54 | 80 | 106 | 132 | 158 |
+-----------+---+----+----+----+-----+-----+-----+
| 38        | 6 | 32 | 58 | 84 | 110 | 136 | 162 |
+-----------+---+----+----+----+-----+-----+-----+
| 39        | 6 | 26 | 54 | 82 | 110 | 138 | 166 |
+-----------+---+----+----+----+-----+-----+-----+
| 40        | 6 | 30 | 58 | 86 | 114 | 142 | 170 |
+-----------+---+----+----+----+-----+-----+-----+
```

## Vùng timming
Là 2 đường nối 3 vùng finder gồm một ngang và một dọc xen kẽ trắng đen và cách phân biệt là dóng thẳng và ngang của 3 ô finder, để dễ hình dung các bạn xem hình minh họa:

![timing-s](/assets/img/blog/8_timing-s.png)

![timing-l](/assets/img/blog/9_timing-l.png)

## Module tối
Module tối là module đen luôn đặt bên cạnh vùng tiếm kiếm dưới trái. Ví trí của module tối được xác định (8,y) trong đó y được tính bằng công thức:
>	**(4 * V) + 9**
với V là phiên bản của mã QR.

## Vùng format information
Đây là vùng cực kỳ quan trọng của mã QR nó chứa thông tin của mask và cấp độ sửa lỗi của mã QR. Trước khi đi vào vùng này mình muốn giải thích rõ hơn về mask và cấp độ sữa lỗi.

## Cấp độ sửa lỗi
Trong mỗi mã QR đều sẽ tồn tại 1 cấp độ sửa lỗi. Có 4 cấp độ sửa lỗi trong mã QR là:
L (low - thấp) có thể phục hồi nếu mức độ hỏng dưới 7%
M (medium - trung bình) có thể phục hồi nếu mức độ hỏng dưới 15%
Q (quartile - vừa ) có thể phục hồi nếu mức độ hỏng dưới 25%
H (high - cao) có thể phục hồi nếu mức độ hỏng dưới 30%

Các cấp độ này cũng được lưu dưới dạng nhị phân với:

- L - 01

- M - 00

- Q - 11

- H - 10

Sau đó được biến đổi cùng mask và lưu vào vị trí format information. Các cấp độ sửa lỗi này sử dụng kỹ thuật sửa lỗi Reed-Solomon, trong bài viết này mình chỉ xin nói về cấu trúc và thành phần của mã QR còn các thuật toán xin hẹn các bạn trong một bài viết khác (nếu mình có hứng thú tìm hiểu vì cá nhân mình không thích toán lắm 😅).
Tuy nhiên có một lưu ý đó là mức độ sửa lỗi càng cao, dữ liệu lưu trữ càng ít vì mã QR phải dùng vùng Error Correction lớn hơn để dự trữ.
Để biết được mã QR bạn đang dùng cấp đô lưu trữ nào mời các bạn theo dõi hình minh họa:

![10_576px-QR_Format_Information](/assets/img/blog/10_576px-QR_Format_Information.png)
![11_format-layout](/assets/img/blog/11_format-layout.png)


Dựa vào hình trên ta có thể thấy được 2 vị trí được viền đỏ đại diện cho 2 bit cấp độ sửa lỗi và trong hình đó là loại L với mã nhị phân 11.

Còn về mask, đây là một chức năng đổi trắng thành đen và ngược lại, chức năng này nhằm giúp các máy đọc mã QR đọc dễ dàng hơn. Có 8 loại mask và việc dùng loại mask nào cũng như thuật toán mình xin hẹn dịp khác lý do như trên ↑. Còn về cách giải mã mình sẽ đề cập ở dưới.

![12_ss_mask](/assets/img/blog/12_ss_mask.png)

Trong hình minh họa trên dễ thấy rằng mặt dù 2 mã QR cùng một version, cùng một cấp sửa lỗi, và cùng một nội dung chỉ khác mask nhưng chúng ta đã có 2 mã QR khác nhau.

## Vùng version
Từ phiên bản 7 trở đi sẽ có 18 bit quy định phiên bản của mã QR với hình chữ nhật có kích thước 6x3.

![13_version-location-1](/assets/img/blog/13_version-location-1.png)


## Encode và decode trong mã QR
Trong phần này mình sẽ chỉ nói về việc đặt các bit như thế nào còn việc encode của từng kiểu dữ liệu cũng như thuật toán encode mình xin không viết tại đây lý do thì như trên :D
Việc đặt các bit trắng đen trong mã QR sẽ được đặt bắt đầu từ góc dưới phải hướng lên cho đến khi gặp các bit cấu trúc thì rẽ trái hướng xuống cho đến cạnh dưới sẽ rẽ trái và hướng lên cứ tiếp tục như vậy cho đến hết :

![data-bit-progression](/assets/img/blog/14_data-bit-progression.png)

Theo chiều hướng lên các bit sẽ đươc đặt như sau:

![upward](/assets/img/blog/15_upward.png)

![qr-code-first-pixel](/assets/img/blog/16_qr-code-first-pixel.png)

![qr-code-second-pixel](/assets/img/blog/17qr-code-second-pixel.png)

![qr-code-third-pixel](/assets/img/blog/18_qr-code-third-pixel.png)

![qr-code-fourth-pixel](/assets/img/blog/19_qr-code-fourth-pixel.png)

![qr-code-fifth-pixel](/assets/img/blog/20_qr-code-fifth-pixel.png)

Chiều hướng xuống:

![downward](/assets/img/blog/21_downward.png)

![qr-code-down-1](/assets/img/blog/22_qr-code-down-1.png)

![qr-code-down-2](/assets/img/blog/23_qr-code-down-2.png)

![qr-code-down-3](/assets/img/blog/24_qr-code-down-3.png)

![qr-code-down-4](/assets/img/blog/25_qr-code-down-4.png)

![qr-code-down-5](/assets/img/blog/26_qr-code-down-5.png)





Khi gặp alignment:

Các bit sẽ bỏ qua alignment và ghi lên ô phía trên nó.


![alignment-modules1](/assets/img/blog/27_alignment-modules1.png)

![alignment-modules2](/assets/img/blog/28_alignment-modules2.png)

![alignment-modules3](/assets/img/blog/29_alignment-modules3.png)

![alignment-modules4](/assets/img/blog/30_alignment-modules4.png)

![alignment-modules5](/assets/img/blog/31_alignment-modules5.png)

![alignment-modules6](/assets/img/blog/32_alignment-modules6.png)

![alignment-modules7](/assets/img/blog/33_alignment-modules7.png)

![alignment-modules8](/assets/img/blog/34_alignment-modules8.png)

![alignment-modules10](/assets/img/blog/35_alignment-modules10.png)

Khi gặp vùng timing:
Bắt đầu tiếp ở cột bên trái vùng timming

![timing-modules1](/assets/img/blog/36_timing-modules1.png)

![timing-modules2](/assets/img/blog/37_timing-modules2.png)

![timing-modules3](/assets/img/blog/38_timing-modules3.png)

![timing-modules4](/assets/img/blog/39_timing-modules4.png)

Cách decode mã QR:

![280px-QR_Character_Placement](/assets/img/blog/40_280px-QR_Character_Placement.png)



Như đã đề cập ở trên, cách chúng ta decode cũng sẽ giống với cách mà mã QR encode. Tuy nhiên cần chú ý:
Ô Enc biểu thị cách encode trong mã QR, có các kiểu enc:

- Numeric (0001) (10 bit cho 3 số)
- Alphanumeric (0010) (11 bit cho 2 ký tự)
- Byte (0100) (8 bit cho 1 ký tự)
- Kanji (1000) (13 bit cho 1 ký tư)
- Structured append (0011) 

![280px-QR_Ver3_Codeword_Ordering](/assets/img/blog/41_280px-QR_Ver3_Codeword_Ordering.png)

- ECI (0111)
- FNC1 (0101)
- FNC1 (1001) 
- Kết thúc (0000)

Hiện nay kiểu encode thông dụng nhất mà chúng ta hay thấy khi đặt vé, link web,... sử dụng kiểu byte encode vì nó dùng 8 bit và được biểu thị theo bảng mã ascii.

- Có một điểm lưu ý là alphanumeric (chữ hoa và số) chỉ có thể encode chữ hoa và số cùng với một vài kí tự đặt biệt nên rất hạn chế, đó là lý do vì sao kiểu byte vô cùng phổ biến.
Ô Len biểu thị cho ta biết chiều dài của dữ liệu trong mã QR. Tùy vào phiên bản mà ta có số bit lưu trữ tương ứng:

| Encoding     | Ver. 1–9 | Ver. 10–26 | Ver. 27–40 |
|--------------|----------|------------|------------|
| Numeric      |    10    | 	 12    |      14    |
| Alphanumeric |     9    | 	 11    |      13    |
| Byte         |     8    | 	 16    |      16    |
| Kanji        |     8    | 	 10    |      12    |

Đầu tiên để decode chúng ta cần phải có một mã QR, và xác định phiên bản của mã QR đó.

![blue](/assets/img/blog/42_blue.png)

![img_1](/assets/img/blog/43_1.PNG)

Ta có 21x21 => đây là phiên bản 1.

Tiếp theo xác định các thành phần của mã QR gồm finder, timing, dark module, format infomation (mask, cấp sửa lỗi).

![img_2](/assets/img/blog/44_2.PNG)

Trong phần màu nâu tiến hành so sánh 3 ô tại vị trí (8,2), (8,3), (8, 4) ta biết được hình QR trên sử dụng mask 100 là kiểu mask nằm ngang

Sau khi xác định được loại mask sử dụng, chúng ta vẽ mask tương ứng:

![QRCode](/assets/img/blog/45_QRCode_11.jpg)


![img_3](/assets/img/blog/46_3.PNG)

![img_4](/assets/img/blog/47_4.PNG)


_Chú ý trong quá trình vẽ mask các bạn để ý đến vùng timming_

Tiếp theo chồng hình QR lên trên mask

![img_5](/assets/img/blog/48_5.PNG)


Chúng ta biến đổi theo công thức: 

- Trắng giữ nguyên.
- Đen không chồng mask vẫn là đen.
- Đen chồng mask ra trắng.
- Mask thành đen.




Sau khi đã sửa, xác định loại enc và len của mã QR

![img_6](/assets/img/blog/49_6.PNG)


Trong hình trên, mình xác định enc bằng mã bit 0100 là kiểu byte và đây là phiên bản 1 + enc là byte nên len bit sẽ là 8. Như hình thì sau khi đổi chúng ta có chiều dài của dữ liệu sẽ là 4 ký tự. Chắc các bạn thắc mắc tại sao 0-7 lại từ trên xuống dưới thì khi ta vì khi encode 00100000 sẽ viết dãy số từ trái qua phải, từ dưới lên trên trong QR nên ta sẽ decode theo kiểu MSB theo 2 hình hướng dẫn ở bên trên. Về đường việc dữ liệu đi theo đường zigzag ảnh ở trên đã chỉ rõ nhưng mình thêm ảnh này để các bạn dễ hình dung hơn trong việc đặt giá trị:

![img_7](/assets/img/blog/50_7.PNG)



Tiến hành đọc dữ liệu:

![img_8](/assets/img/blog/51_8.PNG)


Công các số có trong bit đen và so sánh với bảng mã ascii trong hệ 10 ta có chữ b. Tương tự ta có các chữ khác trong ảnh QR trên:

![img_8](/assets/img/blog/52_9.PNG)

![img_8](/assets/img/blog/53_10.PNG)

![img_8](/assets/img/blog/54_11.PNG)

![img_8](/assets/img/blog/55_12.PNG)

![img_8](/assets/img/blog/56_13.PNG)

Vậy ảnh QR chứa chữ blue hoàn toàn khớp so với dữ liệu  ảnh ban đầu của chúng ta.

**Peace!**
