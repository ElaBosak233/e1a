---
title: 台州学院 2023 baigei_flag
date: 2023-10-24
categories:
  - CTF
  - Misc
---

## 前置工作

下载附件 `scene.bmp`，直接打开来是一张风景画

![](https://z1.ax1x.com/2023/10/27/piZgmcT.png)

用普通的图片隐写解密方法都没用，发现把图片放大有很多零散的像素点

![](https://z1.ax1x.com/2023/10/27/piZg1E9.png)

发现这些像素点每隔 30 个像素就有一个，起点是 (15,15)，我们想办法提取这些像素点

## 编写脚本

```python
from PIL import Image

# 读取原始图片
original_image = Image.open("scene.bmp")
original_width, original_height = original_image.size

# 创建特殊图片
special_image = Image.new("RGB", (original_width, original_height))

# 逐像素复制颜色值
for y in range(15, original_height, 30):
    for x in range(15, original_width, 30):
        pixel = original_image.getpixel((x, y))
        # 将提取到的像素点放大
        for i in range(15):
            for j in range(15):
                special_image.putpixel((x + i, y + j), pixel)

# 保存特殊图片
special_image.save("scene2.bmp")

```

最后得到的图片 `scene2.bmp` 如此

![](https://z1.ax1x.com/2023/10/27/piZgWDg.png)