---
title: BJDCTF2020 The mystery of ip
date: 2023-11-08
categories:
  - CTF
  - Web
---

## 前置工作

对于模板的判断可以参考前天写的 `[BJDCTF2020] Cookie is so stable`，同一个比赛，同一个平台，模板是 Twig，不多解释

开始找注入点，进入 Flag 页面，看到这样的内容

![](https://z1.ax1x.com/2023/11/08/pi1R2Qg.png)

注入点应当为网页获取我们 IP 的位置

## X-Forwarded-For

这或许是一个常识，对于我而言，所以很轻松就能拿下

![](https://z1.ax1x.com/2023/11/08/pi1WilD.png)