---
title: NewStarCTF 2023 AndroGenshin
date: 2023-10-24
categories:
  - CTF
  - Reverse
---

## 前置工作

下载附件 `genshin_impact.apk`，用 Jadx 打开，找到主类 `MainActivity`

![](https://z1.ax1x.com/2023/10/14/pipIYo6.png)

很明确，要解决的就是那一串密码 `YnwgY2txbE8TRyQecyE1bE8DZWMkMiRgJW1=`

虽然上去是 Base64，但是他叠甲了，我们得对加密代码进行逆向

## 编写 Base64 解密算法

其实你可以把加密算法丢给 ChatGPT，让他帮你写

```java
public static byte[] decode(String encodedData, String CUSTOM_TABLE) {
        StringBuilder result = new StringBuilder();
        for (int i = 0; i < encodedData.length(); i += 4) {
            int value = 0;
            for (int j = 0; j < 4; j++) {
                char c = encodedData.charAt(i + j);
                if (c == '=') {
                    value <<= 6;
                } else {
                    int index = CUSTOM_TABLE.indexOf(c);
                    value = (value << 6) | index;
                }
            }
            result.append((char) ((value >> 16) & 255));
            if (encodedData.charAt(i + 2) != '=') {
                result.append((char) ((value >> 8) & 255));
            }
            if (encodedData.charAt(i + 3) != '=') {
                result.append((char) (value & 255));
            }
        }
        return result.toString().getBytes(StandardCharsets.ISO_8859_1);
    }
```

## 编写解密主算法

用 Jadx 把引用常量替换成常量，比如 `int[] base64_table` 中的 `ItemTouchHelper.Callback.DEFAULT_SWIPE_ANIMATION_DURATION` 常量可以直接替换成 `250`

由于 RC4 的解密就是再加密一次，所以很容易就完成了主类的编写

```java
public class Main {
    public static void main(String[] args) {
        int[] base64_table = {125, 239, 101, 151, 77, 163, 163, 110, 58, 230, 186, 206, 84, 84, 189, 193, 30, 63, 104, 178, 130, 211, 164, 94, 75, 16, 32, 33, 193, 160, 120, 47, 30, 127, 157, 66, 163, 181, 177, 47, 0, 236, 106, 107, 144, 231, 250, 16, 36, 34, 91, 9, 188, 81, 5, 241, 235, 3, 54, 150, 40, 119, 202, 150};
        String retval = it_is_not_RC4.rc4("genshinimpact", base64_table);
        String result2 = new String(it_is_not_base64.decode("YnwgY2txbE8TRyQecyE1bE8DZWMkMiRgJW1=", retval));
        System.out.println(result2);
    }
}
```

最后运行程序，得到 Flag 为 `flag{0h_RC4_w1th_Base64!!}`