---
title : "Decompile và Recompile ứng dụng APK"
author : "F4c3l3ss_"
date : 2021-04-17
tags : [research, android]
categories: [Research, Mobile, Android]
---

Cách mình patch code và nghịch ngợm 1 file APK.

## 0x01: Các công cụ cần thiết:

Các bạn tải những công cụ:
- [apktool](https://ibotpeaches.github.io/Apktool/)_Công cụ giúp chúng ta decompile từ 1 file APK sang mã nguồn smali và Recompile thành 1 file APK mới.
- keytool_Công cụ giúp tạo certificate.
- jarsigner_Công cụ giúp ký chữ ký được tạo từ keytool.
- zipalign_Công cụ tinh chỉnh nén cho file APK.

## 0x02: Cách cài đặt

Những công cụ được nêu trên đều có thể dễ dàng cài đặt trên các package của Linux. Ở đây mình sử dụng Ubuntu:

```bash
$ sudo apt install -y apktool zipalign default-jdk
```

Trên hệ điều hành Windows thì các bạn phải cài từng gói riêng và thêm path vào Enviroment để các công cụ trên có thể hoạt động tốt nhất.


## 0x03: Decompile

Cách mình thường làm đó là:

```bash
$ apktool d <app_name>.apk
```

Sau đó chỉnh sửa smali code của file apk vừa decompile.

Cách khác giúp bạn nào chưa quen smali code vẫn có thể giúp bạn dễ patch hơn bằng code java, tuy nhiên cách này mình không dùng thường xuyên vì nó phải convert quá nhiều lần dẫn đến app sau khi patch bị lỗi và không thể hoạt động.

```bash
$ apktool d -rs <app_name>.apk
```
Cách này sẽ không decompile mã nguồn sang smali code mà sẽ để ở dạng đuôi `.dex`.
Để đọc và chỉnh sửa dễ dàng hơn, ta cài đặt [dex2jar](https://github.com/pxb1988/dex2jar). Sử dụng [jd-gui](http://java-decompiler.github.io/) để đọc code và dùng jar trong jdk để compile file mới sau khi đã chỉnh sửa và recompile lại file apk.

## 0x04: Recompile

Sau khi đã chỉnh sửa, chúng ta cần đóng gói lại thành 1 file APK mới.

```bash
$ apktool b
```

Tiếp đến cần tạo certificate để có thể kí trước khi xuất bản:
```bash
$ keytool -genkey -v -keystore apk-sign.key -alias alias_name -keyalg RSA -keysize 2048 -validity 10000
```

Khi các bạn tạo cert sẽ có yêu cầu tạo password, cứ nhập gì cũng được nhưng hãy chắc chắn là các bạn nhớ nó vì password sẽ được nhập trong bước kí file APK.

```bash
$ jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore apk-sign.key <new_app_name>.apk alias_name
```

Cuối cùng là dùng zipalign:
```bash
$ zipalign -v 4 my_application.apk my_application-aligned.apk
```

Cheer (☞ﾟヮﾟ)☞☜(ﾟヮﾟ☜)
