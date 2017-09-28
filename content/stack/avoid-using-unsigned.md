---
title: Avoid Using Unsigned
date: 2017-09-22T10:58:50+10:00
tags:
  - c/c++
---

unsigned在c语言中的设计稍微有点尴尬，或者说它被滥用了。

首先说明我认为除了在进行位运算中应该用unsigned，其他时候都应该用signed。

比如说，通常表示一个非负整数的的时候可能想到要用unsigned。但unsigned有一些特性和非负整数非常不一致

- unsigned减unsigned还是unsigned
- unsigned乘或者除一个负的signed还是unsigned（unsigned比signed等级要高）

而非负整数减法显然是不封闭的，非负整数乘或者除一个负数也显然不可能还是一个非负整数。

[这个问答](https://stackoverflow.com/q/7488837)中讨论了for循环应该用unsigned，signed还是size_t之类的。

提到了表示大小的类型我们都应该用系统定义的size_t，但不幸的是它因为历史原因被定义成unsigned，现在被认为是许多bug的来源。

用unsigned，一个非常容易出错的是n到0的循环，比如

> 这样写会死循环

```c
for (size_t i = 10; i >= 0; i--) {
  // do something
}
```

> 正确的做法，但是有一点tricky，可读性也不好

```c
for (size_t i = 10; i--; ) {
  // do something
}
```

> 用int就方便的多

```c
for (int i = 0; i >= 0; i--) {
  // do something
}
```

但是，用size_t这种系统定义的标准类型是有好处的。可读性强，一看就知道这个变量表示的是大小。还有ptrdiff_t，一看就知道表示的是指针之间的差值。
