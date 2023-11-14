---
title: MRCTF2020 PYWebsite
date: 2023-11-14
categories:
  - CTF
  - Web
---

进入靶机网页，从头到脚没有什么特别的，输入框也跟个傻子一样（一开始以为题目 PYWebsite 是指模板注入，结果发现这是个 PHP）

算了，直接看源码，验证步骤竟然直接塞前端了，不敢相信

![](https://z1.ax1x.com/2023/11/14/piYpy1x.png)

这里给出了一个 `flag.php` 的位置，我们直接进去

很朴素的网页而已

![](https://z1.ax1x.com/2023/11/14/piYp54A.png)

他说记录了 IP，还说自己能看见，我觉得是 `X-Forwarded-For`，那么就 HackBar 试一下

![](https://z1.ax1x.com/2023/11/14/piYpjEQ.png)

是不是有那么一点点简单？