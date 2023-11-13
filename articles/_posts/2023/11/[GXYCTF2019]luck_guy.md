---
title: GXYCTF2019 luck_guy
date: 2023-11-13
categories:
  - CTF
  - Reverse
---

## 前置工作

用 IDA 64 打开，找到关键函数 `get_flag()`，伪代码如下

```c
unsigned __int64 get_flag()
{
  unsigned int v0; // eax
  int i; // [rsp+4h] [rbp-3Ch]
  int j; // [rsp+8h] [rbp-38h]
  __int64 s; // [rsp+10h] [rbp-30h] BYREF
  char v5; // [rsp+18h] [rbp-28h]
  unsigned __int64 v6; // [rsp+38h] [rbp-8h]

  v6 = __readfsqword(0x28u);
  v0 = time(0LL);
  srand(v0);
  for ( i = 0; i <= 4; ++i )
  {
    switch ( rand() % 200 )
    {
      case 1:
        puts("OK, it's flag:");
        memset(&s, 0, 0x28uLL);
        strcat((char *)&s, f1); // GXY{do_not_
        strcat((char *)&s, &f2);
        printf("%s", (const char *)&s);
        break;
      case 2:
        printf("Solar not like you");
        break;
      case 3:
        printf("Solar want a girlfriend");
        break;
      case 4:
        s = 0x7F666F6067756369LL;
        v5 = 0;
        strcat(&f2, (const char *)&s);
        break;
      case 5:
        for ( j = 0; j <= 7; ++j )
        {
          if ( j % 2 == 1 )
            *(&f2 + j) -= 2;
          else
            --*(&f2 + j);
        }
        break;
      default:
        puts("emmm,you can't find flag 23333");
        break;
    }
  }
  return __readfsqword(0x28u) ^ v6;
}
```

## 顺序分析

由上代码可知，Flag 已经由 f2 字符串决定了，我们只需要复现 f2 的生成方式即可

首先得先给 f2 赋初始值，即 `case 4` 中的 `strcat(&f2, (const char *)&s);`，因为一开始 f2 是空的，所以这么写就相当于赋值了

然后再是对 f2 进行处理，即 `case 5` 中的循环

最后就是将 Flag 连着 f2 一起输出

那么重点就在于前两步

## 复现 f2 的构造

这里由于大小端序问题，正向读取 s 行不通，反向读取 s 才有结果

```python
s = [0x7F, 0x66, 0x6F, 0x60, 0x67, 0x75, 0x63, 0x69]  # 这里对应着给 f2 赋值
s.reverse()
flag = ""

for i in range(8):  # 这里对应着对 f2 进行的处理
    if i % 2 == 1:
        flag += chr(s[i]-2)
    else:
        flag += chr(s[i]-1)

print(flag)

```

最后得出全部 Flag 为 `GXY{do_not_hate_me}`，上传到 Buu 里的时候记得把 `GXY` 改成 `flag`