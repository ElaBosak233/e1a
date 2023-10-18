---
tags:
  - BuuCTF
  - 万里挑一
  - DTMF
---

# \[NewStarCTF 2023\] 滴滴滴

## 前置工作

下载附件，发现两个文件

```
奇怪的音频.wav
secret.jpg
```

用 Audacity 打开 `奇怪的音频.wav`，发现在波谱中没有什么隐藏的，可能内容就是音频本身，听一下，感觉有点像电话号码拨打的声音，但这种信息叫做 **DTMF**，于是需要使用一个工具，叫做 `DTMF2NUM`，这里是他的 [Github](https://github.com/Moxin1044/DTMF2NUM) 链接

## DTMF2NUM

下载源码得到 `dtmf2num.exe`，将 `奇怪的音频.wav` 重命名为 `1.wav`，再将音频与 `dtmf2num.exe` 放在同一目录下，打开终端，输入命令

```
.\dtmf2num.exe -w 1.wav
```

随后得出如下信息

```
DTMF2NUM 0.2
by Luigi Auriemma
e-mail: aluigi@autistici.org
web:    aluigi.org

- open 1.wav
  wave size      61600
  format tag     1
  channels:      1
  samples/sec:   8000
  avg/bytes/sec: 16000
  block align:   2
  bits:          16
  samples:       30800
  bias adjust:   0
  volume peaks:  -32766 32767
- dump 1.wav

- MF numbers:    477

- DTMF numbers:  52563319066
```

最后一行的 `DTMF numbers` 就是这段音频的含义了，也是我们后续使用 `steghide` 时用到的解密密码

## steghide

把 `secret.jpg` 和 `steghide` 放在同一目录下，输入如下命令

```
.\steghide.exe extract -sf secret.jpg -p 52563319066
```

输出

```
wrote extracted data to "fffflllllaaaaaggggg.txt".
```

打开 `fffflllllaaaaaggggg.txt`，看到 Flag 就在脸上了

```
flag{1nf0rm4t10n_s3cur1ty_1s_a_g00d_j0b_94e0308b}
```