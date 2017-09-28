---
title: Extra Empty String of Split
date: 2017-09-22T10:58:50+10:00
tags:
  - python
---

> 例子如下

```python
>>> '/segment/segment/'.split('/')
['', 'segment', 'segment', '']
```

这是一个常见的问题，通常我们会期望这个split之后前后是没有empty string的。因为我们split就是为去掉分隔符和没有实际意义的值。

但python中的split就是这样工作的，我也发现scala中的split也是同样。说明大多数语言里可能都是这样实现。

那为什么呢？[一种解释](https://stackoverflow.com/q/2197451)是为了能够还原原来的字符串。

```python
>>> '/'.join(['', 'segment', 'segment', ''])
'/segment/segment/'
```

> 对比没有empty string的结果

```python
>>> '/'.join(['segment', 'segment'])
'segment/segment'
```

然而，我们在使用regexp的时候这种说法就没什么意义。因为一个分隔符可能对应多个字符串。[python官方文档](https://docs.python.org/3/library/re.html#re.split)的解释是

> If there are capturing groups in the separator and it matches at the start of the string, the result will start with an empty string. The same holds for the end of the string

> That way, separator components are always found at the same relative indices within the result list.

这个不是很好理解。但直观来说，如果有n个分隔符，我们会期望结果数组的长度是n+1。这样定义可以保证函数的一致性。所以，如果头尾有分隔符，补足empty string也是为了保持这个一致性。

按照这样的定义，我们可以清楚的知道下面代码的结果是正确的。（如果不是这样定义，则下面代码的结果很难确定中间是否会有那个空格）

```python
>>> 'a//b'.split('/')
['a', '', 'b']
```
