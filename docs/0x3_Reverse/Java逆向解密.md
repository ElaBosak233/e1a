---
tags:
  - BuuCTF
  - 新手
  - Java
---

# Java 逆向解密

> 程序员小张不小心弄丢了加密文件用的秘钥，已知还好小张曾经编写了一个秘钥验证算法，聪明的你能帮小张找到秘钥吗？

## 前置工作

下载附件 `Reverse.class`，用 Jadx 打开，直接找主类

```java
package defpackage;

import java.util.ArrayList;
import java.util.Scanner;

/* renamed from: Reverse  reason: default package */
/* loaded from: Reverse.class */
public class Reverse {
    public static void main(String[] args) {
        Scanner s = new Scanner(System.in);
        System.out.println("Please input the flag ：");
        String str = s.next();
        System.out.println("Your input is ：");
        System.out.println(str);
        char[] stringArr = str.toCharArray();
        Encrypt(stringArr);
    }

    public static void Encrypt(char[] arr) {
        ArrayList<Integer> Resultlist = new ArrayList<>();
        for (char c : arr) {
            int result = (c + '@') ^ 32;
            Resultlist.add(Integer.valueOf(result));
        }
        int[] KEY = {180, 136, 137, 147, 191, 137, 147, 191, 148, 136, 133, 191, 134, 140, 129, 135, 191, 65};
        ArrayList<Integer> KEYList = new ArrayList<>();
        for (int i : KEY) {
            KEYList.add(Integer.valueOf(i));
        }
        System.out.println("Result:");
        if (Resultlist.equals(KEYList)) {
            System.out.println("Congratulations！");
        } else {
            System.err.println("Error！");
        }
    }
}
```

他有了加密算法，那么我们只需要写出解密算法即可

## 编写逆向

```java
import java.util.ArrayList;

public class Main {
    public static void main(String[] args) {
        int[] KEY = {180, 136, 137, 147, 191, 137, 147, 191, 148, 136, 133, 191, 134, 140, 129, 135, 191, 65};

        ArrayList<Character> result = new ArrayList<>();
        for (int i : KEY) {
            int decrypted = (i ^ 32) - '@';
            char c = (char) decrypted;
            result.add(c);
        }

        StringBuilder sb = new StringBuilder();
        for (char c : result) {
            sb.append(c);
        }
        System.out.println(sb);
    }
}

```

最后得出的 Flag 为 `flag{This_is_the_flag_!}`