---
title: "Radix Conversion"
date: 2017-10-20T19:31:13+11:00
---

### 题目

输入一个整数(int类型)`n`以及进制`r(r>=2)`，把整数转换为相应进制的字符串。

结果和python中`hex() oct() bin()`类似

__Example 1:__

`n = 10, r = 2`

Answer: `1010`

__Example 2:__

`n = 123, r = 16`

Answer: `7B`

### 题目2

再实现令一个函数把相应进制的字符串转换为整数。和python中`int(s, r)`类似

__Example 1:__

`s = '1010', r = 2`

Answer: `10`

__Example 2:__

`s = '7B', r = 16`

Answer: `123`

### 实现

```python
def to_radix_str(n, r):
    result = []
    while n:
        d = n % r
        if d >= 10:
            s = chr(65 + d - 10)
        else:
            s = str(d)
        result.append(s)
        n //= r
    return ''.join(reversed(result))

def from_radix_str(s, r):
    result = 0
    for c in s:
        result = r * result + int(c)
    return result
```
