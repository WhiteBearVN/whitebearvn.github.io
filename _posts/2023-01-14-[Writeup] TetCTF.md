---
title: "[Writeup] TetCTF 2023 "
author: "Bu"
date: 2023-01-13
categories: [Writeup,TetCTF2023]
tags: [writeup,web,ctf]
math: true
mermaid: true
---

Trong bài này anh Tsu cho chúng ta một file [code](/file/tetctf2023/newyearbot.py):

Phân tích source code ta thấy bot sẽ querry theo các thông tin có sẵn trong các array và in ra theo điều kiện đầu vào. Để có được flag, ta cần phải làm thế nào để in ra được nội dung của biến Fl4G được lấy từ biến môi trường.

Tổng quan sau khi review toàn bộ file source dynamic test chúng ta có hướng để khai thác bằng cách:
 - Lợi dụng các array NewYear... đang được setup là biến global cùng với FL4G -> ta có thể gọi biến này bằng tham số type khi thực hiện POST request.
 - Vì code đang cho chạy debug nên khi ta truy cập web với url /?debug=Bu, ta có thế biết được chiều dài của FL4G.
 - Lợi dụng 
 ```python
    else:
	    greeting = eval("%s[%s]" % (greetType, greetNumber))
 ```
để in ra vị trí của từng kí tự trong mảng, mà ở đây cụ thể là giá trị của từng vị trí trong mảng FL4G.

Tuy nhiên, challenge này không dễ như vậy vì chúng ta có hàm (**botValidator**) sẽ kiểm tra giá trị number đầu vào, và so sánh với chiều dài tối đa của các array NewYear - 1, và chúng ta có 5. Như vậy với mảng FL4G bao gồm 24 kí tự, ta có thể gọi được phần tử 0, 1, 2, 3, 4, 5 và 24, 23, 22, 21 (-5, -4, -3, -2, -1).

Vậy làm sao chúng ta có thể in ra được toàn bộ flag?
Để ý rằng, trong code, hàm (**botValidator**) cần kiểm tra số, nhưng lại cho range kí tự được nhập là từ 0 - 57 và 124 - 255. Điều này giúp chúng ta có thể truyền vào những toán tử đặc biệt vì vị trí trong mảng có thể thực hiện phép toán.

Tới đây thì ta chỉ việc tính toán và thực hiện phép tính sao cho tham số truyền vào sẽ không lớn 6 sau khi loại bỏ các kí tự.
Để logic và có thể đọc được toàn bộ các vị trí thì mình đã dùng toán tử NOT ```~```

Ta có:

```
~0 = -1 => (~0+~0)*5 = -10
            ~0+~5    = -7

```

Cứ tiếp tục thục hiện các phép toán tương ứng với các vị trí còn thiếu ta sẽ có được flag.

Peace!


