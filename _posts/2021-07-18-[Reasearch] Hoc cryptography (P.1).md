---
title : "[Research] Học cryptography (P.1)"
author : "F4c3l3ss_"
date : 2021-07-18
tags : [research, cryptography]
categories: [Research, Cryptography]
---

Dịch giặc hoành hành, nên mình có thời gian thêm ở nhà làm việc và research một vài thứ mới, hay ho và để bổ sung kiến thức. Mã hóa là một phần của CTF và ATTT; nên mình quyết định học crypto làm crypto viết crypto, một mảng mà mình chưa bao giờ đụng tới trước giờ.

## 0x01: Mật mã học

Mật mã có 2 dạng là mật mã hiện đại và mật mã cổ điện; hiện tại thì mình tập trung nhiều hơn về cách thức mã hóa và giải mã cho mật hiện đại. Các dạng mật mã cổ điển như Rot, Vigenere cipher mình sẽ không tập trung hoặc sẽ tập trung trong phần khác.

Mật mã hiện đại có thể kể đến những thuật toán nổi tiếng bao gồm cả mã hóa đối xứng và mã hóa bất đối xứng như MD5, SHA, RSA, DES, AES,...

## 0x02: Mã hóa đối xứng và bất đối xứng

### Mã hóa đối xứng (Symmetric-key cryptography)

Có thế nói đây là kỹ thuật mã hóa đơn giản nhất thực hiện nhanh nhất.
Kỹ thuật này dùng chung một key (khóa) để mã hóa và giải mã.
Mã hóa đối xứng phổ biến là Rot, Vigenere, AES, DES.

### Mã hóa bất đối xứng (Asymmetric-key Cryptography)

Kỹ thuật mã hóa phức tạp và vì thế sẽ tốn thời gian để mã hóa, giải mã.
Kỹ thuật này sẽ sử dụng 2 khóa, khóa công khai (public key), khóa bí mật (private key).
Khi một thông tin được mã hóa bởi khóa công khai thì chỉ có thể dùng khóa bí mật để giải mã, nhưng thông tin mã hóa bằng khóa bí mật thì có thể dùng khóa công khai để giải mã.

## 0x03: Học từ toán

Yeah, mật mã học về cơ bản là những thuật toán sử dụng toán và lý thuyết số để thực hiện việc mã hóa, giải mã.

### Ước chung lớn nhất (GCD)

- Ước số: Nếu có số tự nhiên a chia hết cho b, thì ta nói b là ước của a.
Ví dụ: 18 % 6 = 0 => 6 là ước của 18

- Ước chung lớn nhất của hai hay nhiều số nguyên là số nguyên dương lớn nhất thuộc tập ước chung các số đó.
Ví dụ: UCLN(18,30) = 6

### Số nguyên tố

- Là số chỉ chia hết cho 1 và chính nó:
Ví dụ: 2, 3, 5, 7, 11, 13,...

### Số nguyên tố cùng nhau

- Hai số nguyên tố cùng nhau là 2 số có UCLN là 1.

Ví dụ: 9 và 10; 15 và 37

### Số đồng dư

- Cho 2 số a, b. Nếu a và b chia cho N có cùng số dư k thì ta gọi a và b là 2 số đồng dư theo n.
Kí hiệu a ≡ b mod N
Ví dụ 14 ≡ 2 mod 12, vì 14 % 12 = 2 và 2 % 12 = 2.

Khi a ≡ b mod N, ta sẽ luôn tồn tại một số k sao cho:
a - b = kn
Vì vậy:
a = kn + b

Xét ví dụ 14 ≡ 2 mod 12
Ta có: 
14 - 12 = k.12 => k = 1
a = kn + b <=> 14 = 1*12 + 2 (điều này đúng)

Phần một đến đây thôi, phần 2 mình sẽ tập trung chủ yếu hơn về số nguyên tố và số đồng dư cùng các tính chất.

Cheer (☞ﾟヮﾟ)☞ ☜(ﾟヮﾟ☜)



