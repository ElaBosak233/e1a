---
title: ZJCTF 2023 ida_pro
date: 2023-11-13
categories:
  - CTF
  - Reverse
---

## 前置工作

这道题是浙江省赛决赛的逆向签到题目，因为对 Reverse 不熟悉，所以我没做出来，是一天后向学长请教才知道怎么搞的

先用 ExeInfoPE 查壳，发现是 UPX

![](https://z1.ax1x.com/2023/11/13/piGO43D.png)

然后就是 `upx -d ida_pro.exe`，结果发现行不通

```bash
                       Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2023
UPX 4.2.1       Markus Oberhumer, Laszlo Molnar & John Reiser    Nov 1st 2023

        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
upx: .\ida_pro.exe: CantUnpackException: file is modified/hacked/protected; take care!!!

Unpacked 0 files.
```

这种就是 UPX 特征被篡改过了，用 010 Editor 打开看一下

![](https://z1.ax1x.com/2023/11/13/piGOHHI.png)

## 修复 UPX 特征

问题出在这里

![](https://z1.ax1x.com/2023/11/13/piGOqEt.png)

有人把 UPXX 改成了 OPXX，所以改回来就行了

然后再 `upx -d ida_pro.exe`，就能脱壳了，用 IDA 打开脱壳后的程序

![](https://z1.ax1x.com/2023/11/13/piGOjC8.png)

按照提示，我们需要从 Strings，Functions 和 Xref 中找 Flag 的缺失部分

## 从 String 中找 Flag

Shift+F12，一眼就看到了第一段 `DASCTF{Wow`

## 从 Functions 中找 Flag

翻最左边的 Functions 栏目，一下就看到了

```c
int _Comp1et3ly_Uns7and_(void)
{
  printf("you are clear!");
  return 114514;
}
```

第二段就是 `_Comp1et3ly_Uns7and_`

## 从 Xref 中找 Flag

这里说 `Find out which function refer to me!`，结果发现就是下面这个 `sub709()` 函数

![](https://z1.ax1x.com/2023/11/13/piGX9Ds.png)

进入 `sub709()` 函数

```c
int sub709(void)
{
  putchar(104);
  putchar(48);
  putchar(119);
  putchar(95);
  putchar(116);
  putchar(48);
  putchar(95);
  putchar(117);
  putchar(115);
  putchar(51);
  puts("Find out which function refer to me!");
  return 114514;
}
```

把这些十进制转换成字符即可，得出第三段 Flag 是 `h0w_t0_us3`

最后就能得出总 Flag 为 `DASCTF{Wow_Comp1et3ly_Uns7and_h0w_t0_us3_988b8cc45bcb}`