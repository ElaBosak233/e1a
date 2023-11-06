---
title: ACTF2020 新生赛 BackupFile
date: 2023-10-24
categories:
  - CTF
  - Web
---

## 寻找 Backup 文件

进入靶机网页，就一句话

```
Try to find out source file!
```

结合题目 **BackupFile**，可知，我想找到一个备份的文件，从而得到当前页面的源码，通常根页面的文件名是 `index.php`，那么就猜一下，备份文件应该就是 `index.php.bak`，没想到还真给我猜对了（正常情况我觉得应该先用御剑或者 dirsearch 扫一下），打开 `index.php.bak`

```php
<?php
include_once "flag.php";

if(isset($_GET['key'])) {
    $key = $_GET['key'];
    if(!is_numeric($key)) {
        exit("Just num!");
    }
    $key = intval($key);
    $str = "123ffwsfwefwf24r2f32ir23jrw923rskfjwtsw54w3";
    if($key == $str) {
        echo $flag;
    }
}
else {
    echo "Try to find out source file!";
}
```

## 弱比较构建内容

一目了然，他要我通过 GET 给他提供一个 `key`，这个 key 必须是数字，并且转为整数类型后，弱比较等于那一大串字符

!!! note "弱比较"
    如果一个数字和字符串进行比较，则有效字符串（或者字符串开始的合法字符）会被转换成数值并且比较按照数值来进行，否则其值为 0

既然是弱比较，谜底就在谜面上，我们取 `$str` 的前三位即可，即让我们的 `$key` = `123`，答案就出来了，最后的 Flag 是 `flag{be7bfc83-5ce6-44c1-8237-f997598f7b35}`