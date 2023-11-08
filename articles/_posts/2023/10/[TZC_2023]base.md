---
title: 台州学院 2023 baaaaaaaaaaaaaaaaaase
date: 2023-10-24
categories:
  - CTF
  - Misc
tags:
  - 嵌套 Base
---

## 前置工作

下载附件 `base.txt`（很大，36.8 MB），粗略的看看末尾

![](https://z1.ax1x.com/2023/10/26/piZmX7t.png)

先丢到 CyberChef 里，发现不太好搞，是个嵌套的 Base 加密

## 编写脚本

嵌套 Base 加密也没关系，直接 Python 写个脚本完事

```python
import base64


class BaseDecoder:
    def __init__(self, s):
        self.s = s.encode()

    def decode_once(self):
        encodings = ["Base16", "Base32", "Base64", "Base85"]
        cur_encoding = 0
        while True:
            try:
                decoded = base64.b16decode(self.s) if cur_encoding == 0 else \
                    base64.b32decode(self.s) if cur_encoding == 1 else \
                        base64.b64decode(self.s) if cur_encoding == 2 else \
                            base64.b85decode(self.s)
                print(f"{encodings[cur_encoding]}: {decoded.decode()}")
                return decoded
            except Exception:
                cur_encoding += 1
                if cur_encoding == len(encodings):
                    print("Done!")
                    return None
                continue

    def decode(self):
        decoded = self.decode_once()
        while decoded is not None and decoded != self.s:
            self.s = decoded
            decoded = self.decode_once()


with open("base.txt", "r") as f:
    p = f.readlines()
decoder = BaseDecoder(p[0])
decoder.decode()

```