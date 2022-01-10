---
title: "[Writeup] Mates SS3 Round 2"
author: F4c3l3ss_
date: 2018-12-15
categories: [Writeup,MatesCTF]
tags: [writeup,forensics,ctf]
math: true
mermaid: true
---

Chúng ta có 2 file, một file dump và 1 file Login Data của Chrome. Btc yều cầu nhập mật khẩu của facebook -> flag là password của account được lưu trong file dump. Sau khi dùng volatility không thành công mình để ý đến header thì nó là file minidump mà dạng file này thì dùng mimikatz là best. Bài này cũng hên là trong giải OtterCTF vừa rồi ngoài cách mình viết trong writeup thì có cách khác là dùng mimikatz như plugin của volatility nên mình có tìm hiểu.

Để analyze file thì mình load file và xem thử password:

```cmd
mimikatz # sekurlsa::minidump dump.dmp
Switch to MINIDUMP : 'dump.dmp'

mimikatz # sekurlsa::logonpasswords
Opening : 'dump.dmp' file for minidump...

Authentication Id : 0 ; 370922 (00000000:0005a8ea)
Session           : Interactive from 1
User Name         : Admin
Domain            : TERMINALATOR
Logon Server      : TERMINALATOR
Logon Time        : 12/14/2018 11:35:09 AM
SID               : S-1-5-21-1143486773-2803761100-3357382775-1000
        msv :
         [00000003] Primary
         * Username : Admin
         * Domain   : TERMINALATOR
         * NTLM     : 7f59db8bb7564b0774aae7ccb7ca594b
         * SHA1     : caa5c83a1f28c9e51aeb5727adea95c4fd2f5dfa
         [00010000] CredentialKeys
         * NTLM     : 7f59db8bb7564b0774aae7ccb7ca594b
         * SHA1     : caa5c83a1f28c9e51aeb5727adea95c4fd2f5dfa
        tspkg :
        wdigest :
         * Username : Admin
         * Domain   : TERMINALATOR
         * Password : KeepGoingYouAreOnTheRightTrack<3
```

Ok vậy là mình đúng, tuy nhiên thì mình lại fail ở bước này vì mình cứ nghĩ config một máy ảo làm sao cho đúng password và username để unblob được file database. Sau mấy tiếng search hơn chục page google và stackoverflow, cuối cùng mình mới chắc chắn rằng chỉ có thể decrypt trên chính máy tạo ra file database.
Quay lại dùng minikatz mình search google thì thấy đươc một bức ảnh từ twitter của tác giả:

![IMG](/assets/img/blog/matesss3r2.jpg)

Ok, vậy là mimikatz có thể sử dụng để decrypt được chrome password thử dùng như tác giả nhưng chỉ show ra được mỗi gmail đăng nhập. Đọc kỹ wiki và cách decrypt chrome mình rút ra đươc phải load được dpapi (windows data protection) trước thì mới decrypt được password đã bị blob.

```cmd
mimikatz # sekurlsa::dpapi

Authentication Id : 0 ; 370922 (00000000:0005a8ea)
Session           : Interactive from 1
User Name         : Admin
Domain            : TERMINALATOR
Logon Server      : TERMINALATOR
Logon Time        : 12/14/2018 11:35:09 AM
SID               : S-1-5-21-1143486773-2803761100-3357382775-1000
         [00000000]
         * GUID      :  {bb6d3aae-4e07-4a9a-9888-e4ed2c68a837}
         * Time      :  12/14/2018 11:58:33 AM
         * MasterKey :  641bb06ded629dff861f505d9c39aa550a394f021cf20d1966848f990c3a1b6759382115a785409553488227b6fb8abd80baca563a7fbf91afd0a807a3026454
         * sha1(key) :  7f2a0c0502063ca113bd608dc6abc52b443c7739


Authentication Id : 0 ; 370880 (00000000:0005a8c0)
Session           : Interactive from 1
User Name         : Admin
Domain            : TERMINALATOR
Logon Server      : TERMINALATOR
Logon Time        : 12/14/2018 11:35:09 AM
SID               : S-1-5-21-1143486773-2803761100-3357382775-1000


Authentication Id : 0 ; 997 (00000000:000003e5)
Session           : Service from 0
User Name         : LOCAL SERVICE
Domain            : NT AUTHORITY
Logon Server      : (null)
Logon Time        : 12/14/2018 11:34:35 AM
SID               : S-1-5-19


Authentication Id : 0 ; 996 (00000000:000003e4)
Session           : Service from 0
User Name         : TERMINALATOR$
Domain            : SKAINET
Logon Server      : (null)
Logon Time        : 12/14/2018 11:34:35 AM
SID               : S-1-5-20


Authentication Id : 0 ; 57144 (00000000:0000df38)
Session           : UndefinedLogonType from 0
User Name         : (null)
Domain            : (null)
Logon Server      : (null)
Logon Time        : 12/14/2018 11:34:34 AM
SID               :


Authentication Id : 0 ; 999 (00000000:000003e7)
Session           : UndefinedLogonType from 0
User Name         : TERMINALATOR$
Domain            : SKAINET
Logon Server      : (null)
Logon Time        : 12/14/2018 11:34:34 AM
SID               : S-1-5-18
         [00000000]
         * GUID      :  {6dbca713-4f3e-4a53-b01a-ec14ee8391eb}
         * Time      :  12/14/2018 11:35:14 AM
         * MasterKey :  a9c38f628c55d302805671060ed1d0ac9562ab3f642287afc98f2d1a5bd1d4a3aa64934badfdf34c0800a506b5c505fa84436cd24822d498d9cd7b1784497561
         * sha1(key) :  6df946f6c45b9b9ea97138d1d0bde6a7b2b70659
         [00000001]
         * GUID      :  {f22e410f-f947-4e08-8f2a-8f65df603f8d}
         * Time      :  12/14/2018 11:34:35 AM
         * MasterKey :  19c05880b67d50f8231cd8009836e3cdc55610e4877f8b976abd5ca15600d0e759934324c6204b56f02527039e7fc52a1dfb5296d3381aaa7c3eb610dffa32fa
         * sha1(key) :  b859b2b52e7e49cf5c70069745c88853c4b23487


mimikatz # dpapi::chrome /in:logindata

URL     : https://www.facebook.com/ ( https://www.facebook.com/ )
Username: nyaacate+fakefb@gmail.com
 * volatile cache: GUID:{bb6d3aae-4e07-4a9a-9888-e4ed2c68a837};KeyHash:7f2a0c0502063ca113bd608dc6abc52b443c7739
Password: matesctf{it's_4lm0st_2019_t1m3_2_upgr8_to_w1nd0w$_10}
```

Và mình có được password là flag như suy nghĩ ban đầu.
Flag: matesctf{it's_4lm0st_2019_t1m3_2_upgr8_to_w1nd0w$_10}

Là người first blood được câu này mình khá vui vì dù sao cũng được quà bonus của ban tổ chức 😀 mong là mấy round sau vẫn sẽ có vụ này để anh em thêm phần tranh đua 👍

Còn câu your a lizard, harry  mình chỉ biết nó dùng SMB protocol và có lẽ là lỗi ehernalblue để gửi và nhận cái gì đó, nhưng mò vẫn chưa ra.😁 Đi cổ vũ đội nhà thôi.
Việt Nam vô địch ☝☝☝☝☝
