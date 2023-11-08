---
title: NewStarCTF 2023 Jvav
date: 2023-10-24
categories:
  - CTF
  - Misc
tags:
  - Java 盲水印
---

> 给阿姨来一杯卡布奇诺，多看看题目名哦，秘密就隐藏在图片中

## 前置工作

这种万里挑一的题目有点恶心，题目描述说 *多看看题目名哦*，其实确实在暗示着 Java 盲水印

下载附件，得到 `challenge.png`

![](https://z1.ax1x.com/2023/10/13/pipK9IA.png)

## BlindWatermark

这是一个用 Java 写的盲水印工具，这里是他的 [GitHub](https://github.com/ww23/BlindWatermark) 仓库地址

这里是他的 Usage

```
Usage: java -jar BlindWatermark.jar <commands>
    commands:
        encode <option> <original image> <watermark> <embedded image>
        decode <option> <original image> <embedded image>
    encode options:
        -c discrete cosine transform
        -f discrete fourier transform (Deprecated)
        -i image watermark
        -t text  watermark
    decode options:
        -c discrete cosine transform
        -f discrete fourier transform (Deprecated)
    example:
        encode -ct input.png watermark output.png
        decode -c  input.png output.png
```

然后按照说明，写出下面这条命令

```
java -jar BlindWatermark.jar decode -c challenge.png challenge.png
```

得到 Flag 图片

![](https://z1.ax1x.com/2023/10/13/pipK8MT.png)

最终的 Flag 为 `flag{3bb3c3a628a94c}`