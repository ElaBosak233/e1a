---
title: NewStarCTF 2023 大冤种
date: 2023-10-24
categories:
  - CTF
  - Misc
tags:
  - 汉信码
---

## 前置工作

下载附件，`.gif` 后缀的文件，暗示够明显了，用 `Stegsolve.jar` 打开，发现有一个特别的二维码

![](https://p.sda1.dev/13/a191a24e343b03b0fc51d633726c19b1/piPm6EQ.png)

因为没有足够的芝士储备，我被困扰了很久（甚至用 PS 尝试修复成 QR Code），最后才知道这叫**汉信码**

## 汉信码解码

在[这个网站](https://tuzim.net/hxdecode/)上就能解码，最后得出的 Flag 为 `flag{1_d0nt_k0nw_h0w_to_sc4n_th1s_c0d3_acef808a868e}`