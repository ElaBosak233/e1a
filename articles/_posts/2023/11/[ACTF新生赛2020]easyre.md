---
title: ACTF新生赛2020 easyre
date: 2023-11-28
categories:
  - CTF
  - Web
---

先查壳，发现是 UPX，直接脱壳就行，随后用 IDA 打开主程序

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  _BYTE v4[12]; // [esp+12h] [ebp-2Eh] BYREF
  _DWORD v5[3]; // [esp+1Eh] [ebp-22h]
  _BYTE v6[5]; // [esp+2Ah] [ebp-16h] BYREF
  int v7; // [esp+2Fh] [ebp-11h]
  int v8; // [esp+33h] [ebp-Dh]
  int v9; // [esp+37h] [ebp-9h]
  char v10; // [esp+3Bh] [ebp-5h]
  int i; // [esp+3Ch] [ebp-4h]

  __main();
  qmemcpy(v4, "*F'\"N,\"(I?+@", sizeof(v4));
  printf("Please input:");
  scanf("%s", v6);
  if ( v6[0] != 'A' || v6[1] != 'C' || v6[2] != 'T' || v6[3] != 'F' || v6[4] != '{' || v10 != '}' )
    return 0;
  v5[0] = v7;
  v5[1] = v8;
  v5[2] = v9;
  for ( i = 0; i <= 11; ++i )
  {
    if ( v4[i] != _data_start__[*((char *)v5 + i) - 1] )
      return 0;
  }
  printf("You are correct!");
  return 0;
}
```

注意循环中 `_data_start__` 其实是一个数组，跟进查看可发现内容

```
'}|{zyxwvutsrqponmlkjihgfedcba`_^]\[ZYXWVUTSRQPONMLKJIHGFEDCBA@?>=<;:9876543210/.-,+*)(',27h,'&%'
'$# !"',0
```

优化一下，把 27h 换成 `'`（十六进制的 27 对应着 ASCII 字符的 `'`），所以这个数组应该被写成（在 Python 中）

```python
data_start = '~}|{zyxwvutsrqponmlkjihgfedcba`_^]\[ZYXWVUTSRQPONMLKJIHGFEDCBA@?>=<;:9876543210/.-,+*)(\'&%$# !"'
```

还有值得注意的点，就是 v7 v8 v9，在程序中都被定义成了 int 类型，而每个 int 变量可以存储 4 个 char 变量，也就是一共 12 个字符

再看一下判断语句 `v4[i] != _data_start__[*((char *)v5 + i) - 1]`，实际上可以看成

```python
v4[i] == data_start[ord(flag[i])-1]
```

可以理解成 `ord(flag[i])-1` 就是 `v4[i]` 在 `data_start` 中相同元素的索引值

那么逆过来就是

```python
flag[i] == chr(data_start.find(v4[i])+1)
```

所以我们可以写出解密脚本

```python
v4 = "*F'\"N,\"(I?+@"
flag = ["" for i in range(len(v4))]  # 初始化一个长度为 12 的数组
data_start = '~}|{zyxwvutsrqponmlkjihgfedcba`_^]\[ZYXWVUTSRQPONMLKJIHGFEDCBA@?>=<;:9876543210/.-,+*)(\'&%$# !"'

for i in range(len(v4)):
    flag[i] = chr(data_start.find(v4[i])+1)

print("".join(flag))

```

最后输出结果为 `U9X_1S_W6@T?`，就是 Flag 了，很有意思吧！