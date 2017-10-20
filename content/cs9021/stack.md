---
title: "Stack"
date: 2017-10-20T17:04:36+11:00
---

## 程序内存分布

建议学c语言加操作系统

![typical program memory layout](/img/mem.jpg)

- 编译后的程序本身（代码、数据常量如字符串常量）
- heap堆（所有动态申请的内存，在python中所有对象的实际内容）
- stack栈（函数中的局部变量，每调用一个函数会栈会往下增长，函数退出时释放）

```python
def f():
    a = 0
    b = 1
    c = [1,2,3]

def main():
    a = 'aaa'
    f()
    b = 'bbb'
```

## 应用

### Lab9 Q1 计算表达式

```python
def evaluate(s):
    stack = deque()
    num = 0
    num_pushed = True
    for c in s:
        if '0' <= c <= '9':
            num = 10 * num + int(c)
            num_pushed = False
        else:
            if not num_pushed:
                stack.append(num)
                num = 0
                num_pushed = True
            if c in '([{+-*/':
                stack.append(c)
            elif c in ')]}':
                calculate(stack, c)
    if stack:
        if len(stack) == 1:
            return stack.pop()
        assert False
    return num
```
