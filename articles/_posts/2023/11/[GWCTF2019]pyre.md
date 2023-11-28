---
title: GWCTF 2019 pyre
date: 2023-11-28
categories:
  - CTF
  - Reverse
---

先 `uncompyle6 attachment.pyc > attachment.py`，顺利反编译

```python
# uncompyle6 version 3.9.0
# Python bytecode version base 2.7 (62211)
# Decompiled from: Python 3.7.9 (tags/v3.7.9:13c94747c7, Aug 17 2020, 18:58:18) [MSC v.1900 64 bit (AMD64)]
# Embedded file name: encode.py
# Compiled at: 2019-08-19 21:01:57
print 'Welcome to Re World!'
print 'Your input1 is your flag~'
l = len(input1)
for i in range(l):
    num = ((input1[i] + i) % 128 + 128) % 128
    code += num

for i in range(l - 1):
    code[i] = code[i] ^ code[i + 1]

print code
code = ['\x1f', '\x12', '\x1d', '(', '0', '4', '\x01', '\x06', '\x14', '4', ',', 
 '\x1b', 'U', '?', 'o', '6', '*', ':', '\x01', 'D', ';', '%', '\x13']
# okay decompiling attachment.pyc
```

看了看，需要逆向两个算法，逆向嘛，从后往前做的都是，所以先看

```python
for i in range(l - 1):
    code[i] = code[i] ^ code[i + 1]
```

很简单的异或运算，逆向的方法就是再异或一遍，注意，处理的顺序需要从后往前，所以我们可以在得知 `code` 数组的情况下这么写逆向代码

```python
code = ['\x1f', '\x12', '\x1d', '(', '0', '4', '\x01', '\x06', '\x14', '4', ',', '\x1b', 'U', '?', 'o', '6', '*', ':', '\x01', 'D', ';', '%', '\x13']

for i in range(len(code)-2, -1, -1):
    code[i] = chr(ord(code[i]) ^ ord(code[i + 1]))
```

再看第二个算法

```python
for i in range(l):
    num = ((input1[i] + i) % 128 + 128) % 128
    code += num
```

当时是已知 `input1[i]` 求 num，而现在是已知 `num` 求 `input1[i]`，值得注意的是，此时我们还原出来的 `code` 数组都还是字符串，我们需要先进行预处理，将其转换成整数数组，方便后续依次提取（`code` 中的每个元素都是算法中的 `num`）

```python
code_int = []
for item in code:
    code_int.append(ord(item))
```

此时 `code_int = [71, 88, 74, 87, 127, 79, 123, 122, 124, 104, 92, 112, 107, 62, 1, 110, 88, 114, 72, 73, 13, 54, 19]`

再初始化一个 `flag` 数组备用

```python
flag = [0 for i in range(len(code_int))]
```

然后就是上面算法主体的逆向了，思路如下（里面所有的 `input1` 都应换成 `code_int`）

> 这个地方的思路极其混乱，若有错误请指正！

对于原始表达式 `num = ((input1[i] + i) % 128 + 128) % 128`

首先，我们将最外层的取模运算逆运算，即去掉最外层的取模：`num = (input1[i] + i) % 128 + 128`（你可以这么想：`1%128` 还是等于 1）

然后移项 `num - 128 = (input1[i] + i) % 128`

因为 `code_int` 中所有元素都没有超过 128，所以可以简化表达式为 `num - i - 128 = input1[i]`

由于取模操作不改变结果在 0~127 范围内的值，所以我们可以将最后的 -128 操作忽略掉


可以得出逆向表达式 `input1[i] = (num - i + 128) % 128`

然后就能把 Flag 打印出来了

```python
for item in flag:
    print(chr(item), end="")
```

最终结果是 `GWHT{Just_Re_1s_Ha66y!}`

最后附上总 Exp

```python
code = ['\x1f', '\x12', '\x1d', '(', '0', '4', '\x01', '\x06', '\x14', '4', ',', '\x1b', 'U', '?', 'o', '6', '*', ':', '\x01', 'D', ';', '%', '\x13']

for i in range(len(code)-2, -1, -1):
    code[i] = chr(ord(code[i]) ^ ord(code[i + 1]))

code_int = []
for item in code:
    code_int.append(ord(item))

flag = [0 for i in range(len(code_int))]

for i in range(len(code_int)-1, -1, -1):
    flag[i] = (code_int[i] - i + 128) % 128

for item in flag:
    print(chr(item), end="")

```