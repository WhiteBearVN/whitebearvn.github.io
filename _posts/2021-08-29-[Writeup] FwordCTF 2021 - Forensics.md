---
title : "[Writeup] FwordCTF 2021 - Forensics"
author : "F4c3l3ss_"
date : 2021-08-29
tags : ["forensics","writeup","FwordCTF"]
categories: [Writeup, FwordCTF 2021]
---
## listening?

Chúng ta có đề bài là một file pcap. Sau khi nhìn tổng quan thì tất cả các package đều chỉ chung về việc truy cập `OAUTH2` của google kèm theo đó là các biến `client_secret`; `refresh_token`;...

![IMG](/assets/img/blog/fword2021/2-flag-listening.JPG)

Response trả lỗi 403, yêu cầu SSL vì đây là truy vấn HTTP nên server không đáp ứng yêu cầu này.

Mình thử với burpsuite để kiểm tra tính xác thực của các biến trên.

![IMG](/assets/img/blog/fword2021/7-burpsuite.JPG)

Chú ý đến giá trị `Scope: https://www.googleapis.com/auth/gmail.readonly`. Điều này có nghĩa là các giá trị, token này sẽ cho phép chúng ta đọc được gmail thông qua API của google.

Tìm kiếm trên Internet, mình có được tài liệu đầy đủ: [https://developers.google.com/gmail/api/reference/rest](https://developers.google.com/gmail/api/reference/rest) và video hướng dẫn: [https://www.youtube.com/watch?v=x7uG1-H0aDU](https://www.youtube.com/watch?v=x7uG1-H0aDU).

Sử dụng postman, set up môi trường theo video và có được flag:

![IMG](/assets/img/blog/fword2021/8-flag.JPG)

Flag: `FwordCTF{email_forensics_is_interesting_73489nn7n4891}`

## Crypt

![IMG](/assets/img/blog/fword2021/3-crypt-debai.JPG)

Download: [Link 1](https://drive.google.com/drive/folders/11KSbrlsTUuhPmGRx5uDjyaJvIR-_gBO8?usp=sharing); [Link 2](https://www.dropbox.com/sh/ipxelxzufpya66s/AAAFVi4cL-sdI0Hj9LGbMd8_a?dl=0); [Link 3](https://mega.nz/folder/0tQy0DCS#f5Q2kLogc7KJs4_y74HkEw)

Đề bài yêu cầu tìm 3 giá trị là ngày giờ nhận mail, người gửi và nội dung file đã bị mã hóa. Trước hết thì mình sử dụng volatility xem thử file dump này có thể có được thông tin từ các tiến trình đang chạy hay không.

```bash
$python2 vol.py -f challenge.raw --profile=Win7SP1x64 pslist
Volatility Foundation Volatility Framework 2.6.1
Offset(V)          Name                    PID   PPID   Thds     Hnds   Sess  Wow64 Start                          Exit                          
------------------ -------------------- ------ ------ ------ -------- ------ ------ ------------------------------ ------------------------------
0xfffffa8018d41890 System                    4      0     76      492 ------      0 2021-08-20 15:30:36 UTC+0000                                 
0xfffffa8019506650 smss.exe                236      4      2       29 ------      0 2021-08-20 15:30:36 UTC+0000                                 
0xfffffa8019158060 csrss.exe               308    300      8      338      0      0 2021-08-20 15:30:42 UTC+0000                                 
0xfffffa8018db9840 wininit.exe             356    300      3       76      0      0 2021-08-20 15:30:43 UTC+0000                                 
0xfffffa8018dbe060 csrss.exe               364    348     10      222      1      0 2021-08-20 15:30:43 UTC+0000                                 
0xfffffa801a179060 winlogon.exe            392    348      4      114      1      0 2021-08-20 15:30:43 UTC+0000                                 
0xfffffa801a1b17c0 services.exe            452    356      7      183      0      0 2021-08-20 15:30:45 UTC+0000                                 
0xfffffa801a1b97f0 lsass.exe               460    356      6      511      0      0 2021-08-20 15:30:45 UTC+0000                                 
0xfffffa801a1c4b30 lsm.exe                 468    356     10      138      0      0 2021-08-20 15:30:45 UTC+0000                                 
0xfffffa801a234b30 svchost.exe             560    452     10      340      0      0 2021-08-20 15:30:49 UTC+0000                                 
0xfffffa801a25db30 svchost.exe             624    452      6      239      0      0 2021-08-20 15:30:50 UTC+0000                                 
0xfffffa801a29bb30 svchost.exe             724    452     20      443      0      0 2021-08-20 15:30:51 UTC+0000                                 
0xfffffa801a2b3920 svchost.exe             768    452     19      417      0      0 2021-08-20 15:30:51 UTC+0000                                 
0xfffffa801a2bb660 svchost.exe             792    452     32      908      0      0 2021-08-20 15:30:51 UTC+0000                                 
0xfffffa801a2e1b30 audiodg.exe             884    724      5      125      0      0 2021-08-20 15:30:53 UTC+0000                                 
0xfffffa801a30d890 svchost.exe             964    452     10      252      0      0 2021-08-20 15:30:55 UTC+0000                                 
0xfffffa801a354b30 svchost.exe             284    452     13      346      0      0 2021-08-20 15:30:57 UTC+0000                                 
0xfffffa801a3cb420 spoolsv.exe            1080    452     12      265      0      0 2021-08-20 15:31:01 UTC+0000                                 
0xfffffa801a38bb30 svchost.exe            1112    452     18      313      0      0 2021-08-20 15:31:01 UTC+0000                                 
0xfffffa801a44a340 armsvc.exe             1232    452      4       63      0      1 2021-08-20 15:31:03 UTC+0000                                 
0xfffffa801a4b4350 svchost.exe            1292    452     11      173      0      0 2021-08-20 15:31:05 UTC+0000                                 
0xfffffa80198b44e0 taskhost.exe           1784    452      9      174      1      0 2021-08-20 15:31:38 UTC+0000                                 
0xfffffa801a5e8b30 dwm.exe                1856    768      3       70      1      0 2021-08-20 15:31:38 UTC+0000                                 
0xfffffa801a5f7b30 explorer.exe           1884   1848     32      970      1      0 2021-08-20 15:31:39 UTC+0000                                 
0xfffffa80192f9b30 StikyNot.exe            512   1884      9      138      1      0 2021-08-20 15:31:43 UTC+0000                                 
0xfffffa801a677b30 SSScheduler.ex          832   1884      1       59      1      1 2021-08-20 15:31:44 UTC+0000                                 
0xfffffa801a28a060 SearchIndexer.         1344    452     12      645      0      0 2021-08-20 15:31:46 UTC+0000                                 
0xfffffa801a6cd910 SearchProtocol         1704   1344      6      310      0      0 2021-08-20 15:31:48 UTC+0000                                 
0xfffffa8019160630 notepad.exe             596    440      1       64      1      0 2021-08-20 15:32:29 UTC+0000                                 
0xfffffa8018e85b30 notepad.exe            1724    304      1       64      1      0 2021-08-20 15:32:58 UTC+0000                                 
0xfffffa80198c5870 sppsvc.exe             1852    452      4      141      0      0 2021-08-20 15:33:09 UTC+0000                                 
0xfffffa80198c5340 svchost.exe             304    452     13      317      0      0 2021-08-20 15:33:10 UTC+0000                                 
0xfffffa8019935b30 notepad.exe             840   1800      1       64      1      0 2021-08-20 15:33:10 UTC+0000                                 
0xfffffa801a17e060 notepad.exe             992   1780      1       64      1      0 2021-08-20 15:33:39 UTC+0000                                 
0xfffffa801a333630 notepad.exe            2164   1884      1       60      1      0 2021-08-20 15:34:24 UTC+0000                                 
0xfffffa801a6e0750 SearchFilterHo         2180   1344      4       85      0      0 2021-08-20 15:34:48 UTC+0000                                 
0xfffffa801a351630 WmiPrvSE.exe           2308    560      7      116      0      0 2021-08-20 15:35:09 UTC+0000                                 
0xfffffa801a3df610 DumpIt.exe             2512   1884      2       45      1      1 2021-08-20 15:35:16 UTC+0000                                 
0xfffffa801a3e2840 conhost.exe            2536    364      2       53      1      0 2021-08-20 15:35:16 UTC+0000
```
Để ý thì thấy rất nhiều tiến trình `notepad.exe` đang chạy. Chúng ta có thể dump memory của tiến trình để xem nội dung file mà notepad đang mở, tuy nhiên thì nó sẽ hơi noise và nhiều dữ liệu nên trước tiên thì mình dùng plugin `cmdline` để xem notepad đang mở những file nào và đường dẫn của những file này trên memory.

```bash
$python2 vol.py -f challenge.raw --profile=Win7SP1x64 cmdline
Volatility Foundation Volatility Framework 2.6.1
************************************************************************
System pid:      4
************************************************************************
smss.exe pid:    236
Command line : \SystemRoot\System32\smss.exe
************************************************************************
csrss.exe pid:    308
Command line : %SystemRoot%\system32\csrss.exe ObjectDirectory=\Windows SharedSection=1024,20480,768 Windows=On SubSystemType=Windows ServerDll=basesrv,1 ServerDll=winsrv:UserServerDllInitialization,3 ServerDll=winsrv:ConServerDllInitialization,2 ServerDll=sxssrv,4 ProfileControl=Off MaxRequestThreads=16
************************************************************************
wininit.exe pid:    356
Command line : wininit.exe
************************************************************************
csrss.exe pid:    364
Command line : %SystemRoot%\system32\csrss.exe ObjectDirectory=\Windows SharedSection=1024,20480,768 Windows=On SubSystemType=Windows ServerDll=basesrv,1 ServerDll=winsrv:UserServerDllInitialization,3 ServerDll=winsrv:ConServerDllInitialization,2 ServerDll=sxssrv,4 ProfileControl=Off MaxRequestThreads=16
************************************************************************
winlogon.exe pid:    392
Command line : winlogon.exe
************************************************************************
services.exe pid:    452
Command line : C:\Windows\system32\services.exe
************************************************************************
lsass.exe pid:    460
Command line : C:\Windows\system32\lsass.exe
************************************************************************
lsm.exe pid:    468
Command line : C:\Windows\system32\lsm.exe
************************************************************************
svchost.exe pid:    560
Command line : C:\Windows\system32\svchost.exe -k DcomLaunch
************************************************************************
svchost.exe pid:    624
Command line : C:\Windows\system32\svchost.exe -k RPCSS
************************************************************************
svchost.exe pid:    724
Command line : C:\Windows\System32\svchost.exe -k LocalServiceNetworkRestricted
************************************************************************
svchost.exe pid:    768
Command line : C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted
************************************************************************
svchost.exe pid:    792
Command line : C:\Windows\system32\svchost.exe -k netsvcs
************************************************************************
audiodg.exe pid:    884
Command line : C:\Windows\system32\AUDIODG.EXE 0x2d8
************************************************************************
svchost.exe pid:    964
Command line : C:\Windows\system32\svchost.exe -k LocalService
************************************************************************
svchost.exe pid:    284
Command line : C:\Windows\system32\svchost.exe -k NetworkService
************************************************************************
spoolsv.exe pid:   1080
Command line : C:\Windows\System32\spoolsv.exe
************************************************************************
svchost.exe pid:   1112
Command line : C:\Windows\system32\svchost.exe -k LocalServiceNoNetwork
************************************************************************
armsvc.exe pid:   1232
Command line : "C:\Program Files (x86)\Common Files\Adobe\ARM\1.0\armsvc.exe"
************************************************************************
svchost.exe pid:   1292
Command line : C:\Windows\system32\svchost.exe -k LocalServiceAndNoImpersonation
************************************************************************
taskhost.exe pid:   1784
Command line : "taskhost.exe"
************************************************************************
dwm.exe pid:   1856
Command line : "C:\Windows\system32\Dwm.exe"
************************************************************************
explorer.exe pid:   1884
Command line : C:\Windows\Explorer.EXE
************************************************************************
StikyNot.exe pid:    512
Command line : "C:\Windows\System32\StikyNot.exe" 
************************************************************************
SSScheduler.ex pid:    832
Command line : "C:\Program Files (x86)\McAfee Security Scan\3.11.1664\SSScheduler.exe" 
************************************************************************
SearchIndexer. pid:   1344
Command line : C:\Windows\system32\SearchIndexer.exe /Embedding
************************************************************************
SearchProtocol pid:   1704
Command line : "C:\Windows\system32\SearchProtocolHost.exe" Global\UsGthrFltPipeMssGthrPipe1_ Global\UsGthrCtrlFltPipeMssGthrPipe1 1 -2147483646 "Software\Microsoft\Windows Search" "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT; MS Search 4.0 Robot)" "C:\ProgramData\Microsoft\Search\Data\Temp\usgthrsvc" "DownLevelDaemon" 
************************************************************************
notepad.exe pid:    596
Command line : "C:\Windows\system32\NOTEPAD.EXE" C:\Users\SemahAB\Downloads\Secret content\flag.txt.asc
************************************************************************
notepad.exe pid:   1724
Command line : "C:\Windows\system32\NOTEPAD.EXE" C:\Users\SemahAB\AppData\Roaming\Thunderbird\Profiles\gb1i6asd.default-release\ImapMail\imap.yandex.com\INBOX
************************************************************************
sppsvc.exe pid:   1852
Command line : C:\Windows\system32\sppsvc.exe
************************************************************************
svchost.exe pid:    304
Command line : C:\Windows\System32\svchost.exe -k secsvcs
************************************************************************
notepad.exe pid:    840
Command line : "C:\Windows\system32\NOTEPAD.EXE" C:\Users\SemahAB\AppData\Roaming\Thunderbird\Profiles\gb1i6asd.default-release\ImapMail\imap.yandex.com\Spam
************************************************************************
notepad.exe pid:    992
Command line : "C:\Windows\system32\NOTEPAD.EXE" C:\Users\SemahAB\AppData\Roaming\Thunderbird\Profiles\gb1i6asd.default-release\ImapMail\imap.yandex.com\Trash
************************************************************************
notepad.exe pid:   2164
Command line : "C:\Windows\system32\notepad.exe" 
************************************************************************
SearchFilterHo pid:   2180
Command line : "C:\Windows\system32\SearchFilterHost.exe" 0 504 508 516 65536 512 
************************************************************************
WmiPrvSE.exe pid:   2308
Command line : C:\Windows\system32\wbem\wmiprvse.exe
************************************************************************
DumpIt.exe pid:   2512
Command line : "C:\Users\SemahAB\Downloads\DumpIt.exe" 
************************************************************************
conhost.exe pid:   2536
Command line : \??\C:\Windows\system32\conhost.exe
```

Khá nhiều thông tin hay ho, bao gồm nội dung inbox của email, spam, trash và cùng với đó là file `flag.txt.asc`, đây có thể là mảnh ghép đầu tiên trong flag. Dump file này cho mình kết quả:

```bash
$python2 vol.py -f challenge.raw --profile=Win7SP1x64 filescan | grep ".txt.asc"
Volatility Foundation Volatility Framework 2.6.1
0x000000007fb5d2e0     16      0 R--rw- \Device\HarddiskVolume2\Users\SemahAB\Downloads\Secret content\flag.txt.asc
$python2 vol.py -f challenge.raw --profile=Win7SP1x64 dumpfiles -Q 0x000000007fb5d2e0 -D .
$cat flag.txt.asc
-----BEGIN PGP MESSAGE-----

hQIMA6mGMG+kfeOaAQ/+K8a+2BQVT+Ixk6xJKytj0xbDUqfw7FUQuKLjWBcxddbn
hd8eKmWIswUYpV3PmluCqJE5LRkdpX3Rdu/iK2ZRNyK7fGA3R3r3KFiBn5qKj0S4
D47LqwDE/fpo5YUGt3GH6Cpujv3DrdIpXcn2yFZLBiho5415bcTn2D3qm/H6uz9N
JFzved20D05OxTjj+a4s8Jsf7eroBj7DSIvvbD5/SF85rO1u1iQQHif2Na1CyrXX
sJyAU59iVipWgjo1uoGfD8PqnhJdctnTbxu2oYY6MpnQ6K98WI6hwyo/crwUGJU1
nQ8JKEQrirmcbZLsHb1fzmLDsEf2f0prG1QJXA3a9panz3N+TvIXvoWRXLkLgOcj
KjK3NjZOlnsngcXNfaSIeZelskfCg096cUiEnfUeDDc8CaGFPO4uGnhnVo/bcEFM
AuAiegy0/avdVIOU/Ho2o3ON6cQceYlsZD/Viy6oeNSib1Iyu3FLO1ZQMiaPZ6j3
Ewrfag7mr5AOLM11WtYJRYtQh2oLV5hYsvxylxZOGdIJ05hDj/CdEcxYLQ0cv3xJ
xi4shl99rGgYL2NGALrL5pnjCBDBzAdIhKImlSZmurA+29/8iE5If+NJk86SQhNl
lhSF+jIrApsQkl0cTPoE+GPmA9QicMXeLXXB+iJjDG7MVOMX1aeJROmvGWTOxBTS
swFiIgGBQQHE4cjQh8soViVMP7tOkraCYZRYGTxlrFJiCeI38gk7hx0n0hMnfMyy
lzzAKo8Fvt69q6smjOns0MhFVIfT4bUrb0md7KMXqn/OvtP/L/S9IEKjsMvk7yl6
wrLYGYwBeIgfHqPIML0DxOuqV9WKWPoigazdRVW2hQqyEjqbJRG/GIyfaisMxNlT
O+BIdWAsM3f7+gytkvmFo4lnR+Y7yVOL618HSBhie2SFwGNu
=YEJl
-----END PGP MESSAGE-----
```
Mình cũng không bất ngờ nhiều khi nội dung file đã bị mã hóa vì theo đề bài thì chúng ta cần tìm cách decrypt thông qua mail mà user gửi và nhận. Tiếp tục dump các file INBOX, SPAM, TRASH.

IBOX: Trong phần này mình chú ý đến email của `ctf.user`. Chúng ta có đoạn base64 dài cần decode. Kết quả thu được là file key để giải mã PGP. Nhưng khi mình import thì lại cần passphrase để mở key. Tiếp tục đi tìm nào.

TRASH: Mình hy vọng là user sẽ xóa mail chứa passphrase, tuy nhiên thì file này lại không cho chúng ta thông tin gì.

SPAM: Cho mình một chi tiết rất quan trọng: `But we used the same secret as your EN VAR 345YACCESSTOGENERATEP455`. Mình đã thử rất nhiều lần `345YACCESSTOGENERATEP455` là passphrase nhưng không thành công và khác là stuck. Thì một người em của mình đã tìm ra vì envars là một trong những plugin của volatility.
```bash
$python2 vol.py -f challenge.raw --profile=Win7SP1x64 envars | grep 345YACCESSTOGENERATEP455
Volatility Foundation Volatility Framework 2.6.1
     308 csrss.exe            0x00000000003a1320 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
     356 wininit.exe          0x000000000043a990 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
     364 csrss.exe            0x0000000000181320 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
     392 winlogon.exe         0x000000000007e060 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
     452 services.exe         0x0000000000341320 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
     460 lsass.exe            0x0000000000351320 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
     468 lsm.exe              0x0000000000231320 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
     560 svchost.exe          0x00000000001e1320 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
     624 svchost.exe          0x0000000000311320 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
     724 svchost.exe          0x00000000001e1320 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
     768 svchost.exe          0x0000000000171320 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
     792 svchost.exe          0x0000000000301320 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
     964 svchost.exe          0x0000000000351320 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
     284 svchost.exe          0x00000000001e1320 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
    1080 spoolsv.exe          0x0000000000161320 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
    1112 svchost.exe          0x0000000000361320 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
    1232 armsvc.exe           0x0000000000151320 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
    1292 svchost.exe          0x0000000000371320 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
    1784 taskhost.exe         0x00000000001c1320 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
    1856 dwm.exe              0x0000000000071320 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
    1884 explorer.exe         0x00000000037cd6b0 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
     512 StikyNot.exe         0x00000000003d1320 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
     832 SSScheduler.ex       0x0000000000501320 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
    1344 SearchIndexer.       0x000000000040a010 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
    1704 SearchProtocol       0x00000000001f1320 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
     596 notepad.exe          0x00000000002b1320 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
    1724 notepad.exe          0x0000000000171320 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
    1852 sppsvc.exe           0x0000000000271320 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
     304 svchost.exe          0x000000000021d0e0 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
     840 notepad.exe          0x0000000000361320 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
     992 notepad.exe          0x0000000000221320 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
    2164 notepad.exe          0x0000000000301320 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
    2180 SearchFilterHo       0x00000000002a1320 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
    2308 WmiPrvSE.exe         0x00000000003d1320 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
    2512 DumpIt.exe           0x0000000000081320 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
    2536 conhost.exe          0x00000000003ad7f0 345YACCESSTOGENERATEP455       pPaAsSpPhHrR445533
```

Decrypt thôi nào
```bash
$gpg --import b64.key
$gpg -o flag --decrypt flag.txt.asc
gpg: encrypted with 4096-bit RSA key, ID A986306FA47DE39A, created 2021-08-20
      "SemahBA (Not too far to get what you want) <SemahBA@fword.tech>"
$cat flag
We are so proud that we sent you to give him back his important file. 

The important thing was : 23b2e901f3c3c3827a70589efd046be8
```

OK flag đã gần hoàn thiện, chỉ còn thời gian gửi, mình có thử 1, 2 lần vì time from và date ở INBOX khác nhau và ra được flag hoàn thiện.

Flag: `FwordCTF{08202021-05:03_ctf.user_23b2e901f3c3c3827a70589efd046be8}`

## eLearning

![IMG](/assets/img/blog/fword2021/4-eLearning-debai.JPG)

Download: [Link 1](https://drive.google.com/file/d/1tfcLMAbBWH-KBnkCDAboXAswm5fZuIed/view?usp=sharing); [Link 2](https://mega.nz/file/dwxHXQJa#loOkmUJfr2c1MNoAlue62IJZX4zHpOsW_QrWWdEUdlw); [Link 3](https://www.dropbox.com/s/chp4g30onaumjnm/Challenge2.zip?dl=0)

Đề bài ta một file ảnh nên mình dùng Autospy để trích xuất dữ liệu.

![IMG](/assets/img/blog/fword2021/5-autospy.JPG)

Sau khi để ý ở Program Files có ứng dụng Mailbird, mình google ra được đường dẫn lưu trữ là `\Users\fword\AppData\Local\Mailbird\Store`. Ở đây chúng ta xuất ra được file lưu trữ email là Store.db, sử DB Browser, mình có được thông tin: 

![IMG](/assets/img/blog/fword2021/6-sender.JPG)

Vậy là đã có được thông tin của giảng viên. Phần còn lại mình đã kiếm log của dirsearch và zenmap nhưng không cho thêm kết quả gì. Thử sử dụng lịch sử powershell: `%APPDATA%\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt` ta thu được:
```Powershell
New-Item -ItemType Diretory -Path C:\users\fword\lab01
New-Item -ItemType Directory -Path C:\users\fword\lab01
ls
cd .\lab01\
nmap -sC -sV 192.168.1.9
cd ..\Downloads\dirsearch-master\dirsearch-master\
echo "192.168.1.9          lab01.fword" >> C:\Windows\System32\drivers\etc\hosts
ls
python .\dirsearch.py -u http://lab01.fword -e txt,php 
cd
cd C
cd C:\
cd
cd C
cd C:\
cd
cd C
cd C:\
cd
cd C
cd C:\
cd
cd C
cd C:\
cd
cd C
cd C:\
cd
cd C
cd C:\Users\fword\
wget http://lab01.fword/secret/secret.txt
Rename-Item -Path "c:\Users\fword\secret.txt" -NewName "file.txt"
```

Vậy là có thêm các tools đã dùng: dirsearch-nmap-wget. Tiếp theo ta đọc file.txt ở thử mục Document: `663cd2dfc9418f384d90c89a15319b3d`

Flag: `FwordCTF{SBA_192.168.1.9_dirsearch-nmap-wget_secret.txt_663cd2dfc9418f384d90c89a15319b3d}`

## shshsh

Lại là một thử thách về memory forensics.

Tuy nhiên thử thách này lại không có bất kì process hay thông tin gì khả nghi. Nên mình thử kiểm tra biến môi trường.

```bash
$ python vol.py -f chall.vmem --profile=WinXPSP2x86 envars

....

1552 explorer.exe         0x00010000 k1                             12a45652a154a358452124565487ad0
1552 explorer.exe         0x00010000 iv                             k1
1552 explorer.exe         0x00010000 KT                             AES128-CBC
1552 explorer.exe         0x00010000 LOGONSERVER                    \\CTFCREATORS-DF

....
```

Chúng ta có key và iv của thuật toán mã hóa AES128-CBC; tuy nhiên lại không có tệp mã hóa.

Đoạn này thì mình cũng mò mãi đến lúc sau thì mới thấy bỏ xót 1 phần quan trọng trong các biến môi trường:

```bash
1552 explorer.exe         0x00010000 PATHEXT                        .HAK;.COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;
```

Chỉ duy nhất PATHEXT của explorer là có đuôi mở rộng rất lạ là .HAK; thử tìm xem có bao nhiêu file .HAK nào. 

```bash
$ python vol.py -f chall.vmem --profile=WinXPSP2x86 filescan | grep -i ".HAK"
Volatility Foundation Volatility Framework 2.6.1
0x00000000012403d8      1      0 -W-rw- \Device\HarddiskVolume1\WINDOWS\system32\Handles.HAK
0x000000000198c0d0      2      2 R--rw- \Device\HarddiskVolume1\WINDOWS\system32\Handles.HAK

$ python vol.py -f chall.vmem --profile=WinXPSP2x86 dumpfiles -Q 0x00000000012403d8 -D .

$ strings file.None.0x8144e990.dat 
NDgwYjJmMzM4ZjY0N2M3ZTY1MjQwZTc2MjdjYWRhYmFjNWJkNWZjNjllNWQxZDMzM2ZlYjRlMjNjYzNhYjA5NGZlY2E1ODcxMjA5ODJmODExM2U0NTMzNjcxOThmMDU4MWJlZDNhZWMxNTBkMjg1NDIzYWUxMTFiN2I5NDkzNjExZmFjMmNmYmViYjg5NzkzY2U3MThjM2YxZTJkNWNlMjcwMmE5MWNjNDM4NzZhYjJmZGJjZjc0MmRlODg4M2Q2OTIzYjAwYTYyNTMzMGRhOWRiNjljNmFiYmU0ZjIxY2Y5NGJiZmMxMWU2MWJkZDc0NmVjNDU5OGRkNTQ2ZDk2YQo=
```

Decrypt thôi nào 
![IMG](/assets/img/blog/fword2021/9-shshsh.JPG)

Flag: `FwordCTF{y0u_CAN_Mak3_u5e_of_4ny_proCesS_with_tHe_ri9ht_apProach}`

Cheer (☞ﾟヮﾟ)☞ ☜(ﾟヮﾟ☜)



