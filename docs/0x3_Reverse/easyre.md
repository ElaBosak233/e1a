---
tags:
  - BuuCTF
  - 水题
---

# easyre

水得不能再水了这题，把附件用 IDA 64 打开，然后按 F5 反汇编，得到的伪代码如下

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  int b; // [rsp+28h] [rbp-8h] BYREF
  int a; // [rsp+2Ch] [rbp-4h] BYREF

  _main();
  scanf("%d%d", &a, &b);
  if ( a == b )
    printf("flag{this_Is_a_EaSyRe}");
  else
    printf("sorry,you can't get flag");
  return 0;
}
```

这 Flag 都在脸上了 `flag{this_Is_a_EaSyRe}`