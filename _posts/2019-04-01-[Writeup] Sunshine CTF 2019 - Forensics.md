---
title : "[Writeup] Sunshine CTF 2019 - Forensics"
author : "F4c3l3ss_"
date : 2019-04-01
tags : ["SunshineCTF2019","writeup"]
categories: [Writeup, SunshineCTF2019]
mermaid: true
---

## Golly

It's a code of Golly rle file, when I run a code given I just have a alphabet table:

![IMG](/assets/img/blog/sunshine2019-1.PNG)

Run it and nothing else, I read a rle file document at here. And I know a "$" represents the end of each row and an optional "!" represents the end of the pattern. So I just copy the paragraph after !$$$$ and I got

![IMG](/assets/img/blog/sunshine2019-2.PNG)

Flag: sun{th1s_w0nt_last}

## Castles

Open the file in HXD I saw something like a hint:

`Hey! Mario said something about a hidden key. Hesaid this: F2I and A1S, and that it was in two pieces`

Because this is 001 file so I use FTK Imager to open it. And I found an JPG image of Mario:

![IMG](/assets/img/blog/Sunshine2019-AlmostThere.jpg)

Because it is an JPG file and need a key to open it so I think the right tool would be steghide. Now I need to find a key to have a flag. With the hint above and in slack file in FTK I found something like a password of Castello_Di_Amorosa and Castelo_da_Feira I combined all of it and have a string:

> *AQ273RFGHUI91OLO987YTFGY78IK*

![IMG](/assets/img/blog/sunshine2019-3.PNG)

Flag: flag{7H4NK5_F0R_PL4Y1NG}

## We Will We Will

It's img file so I used Autospy to open it, after do some stuff I realize I have a luks file.

![IMG](/assets/img/blog/sunshine2019-4.PNG)

Luks file is a encrypted image on Linux so I need a password. But I don't have any hint of password so I think need brute force a password. Hashcat is a good tool for this.

`hashcat64.exe -a 0 -m 14600 luks rockyou.txt`

And the password  is: filosofia
Mount it and I have flag is a partition name

```bash 
$sudo cryptsetup open b.luks flag
```

Flag: sun{wrasslin}
