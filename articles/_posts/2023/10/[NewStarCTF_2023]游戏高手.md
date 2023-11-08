---
title: NewStarCTF 2023 游戏高手
date: 2023-10-24
categories:
  - CTF
  - Web
---

## 前置工作

打开靶机网页，看到如下界面

![](https://z1.ax1x.com/2023/10/09/pPvf4KK.png)

~~方法很简单，打到 100000 分后紫砂~~

这种直接开 F12 看一下网页源码

![](https://z1.ax1x.com/2023/10/09/pPvf5DO.png)

发现了有用的 JS 代码 `app_v2.js`

我们进入 `app_v2.js`，发现都是些乱七八糟的游戏代码，感觉没啥卵用

![](https://z1.ax1x.com/2023/10/09/pPvfxr8.png)

## 找到 100000

我们在 JS 代码界面按 Ctrl + F，搜索 100000，发现了有用的东西

```js
function gameover(){
    if(gameScore > 100000){
        var xhr = new XMLHttpRequest();
        xhr.open("POST", "/api.php", true);
        xhr.setRequestHeader("Content-Type", "application/json");
        xhr.onreadystatechange = function() {
        if (xhr.readyState === 4 && xhr.status === 200) {
            var response = JSON.parse(xhr.responseText);
            alert(response.message);
        }
        };
        var data = {
            score: gameScore,
        };
        xhr.send(JSON.stringify(data));
    }
	alert("鎴愮哗锛�"+gameScore);
	gameScore=0;  
	curPhase =PHASE_READY;  
	hero = null;
	hero = new Hero();  	    
}
```

发现他要往 `/api.php` POST 一条数据，而且也只会在 `gameScore > 100000` 的时候触发，这条数据就是上面的 `data` 变量，也就是说，我们只需要按照 JSON 的格式 POST 这么一条数据即可

```json
{
    "score": 110000
}
```

## 编写 Http 请求

于是最终我们的 Http 请求代码如下

```http
POST /api.php HTTP/1.1
Host: fa83b0e0-1558-4a81-b193-46146126ebd0.node4.buuoj.cn:81
User-Agent: Apifox/1.0.0 (https://apifox.com)
Content-Type: application/json
Accept: */*
Host: fa83b0e0-1558-4a81-b193-46146126ebd0.node4.buuoj.cn:81
Connection: keep-alive
Content-Length: 25

{
    "score": 110000
}
```

发送请求后，得到 Flag 为 `flag{7ff501e2-a739-4e88-ad35-ae4d2857ba09}`