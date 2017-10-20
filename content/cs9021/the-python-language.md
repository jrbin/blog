---
title: "Python语言"
date: 2017-10-19T16:36:32+11:00
---

## 列表生成式

```python
# list
array = [i for i in range(10)]
odd = [i for i in array if i % 2 == 1]
array2d = [[0] * 3 for _ in range(3)]

# set
value_set = {i for i in range(10)}
{0, 1, 2} & value_set
{0, 1, 2} | value_set
{0, 1, 2} - value_set
{1, 2, 3}.issubset(value_set)

# dict
table = {str(i-64): i for i in range(65, 91)}
sorted_pairs = sorted(table.items(), key=lambda x: x[1])
```

## files

```python
with open(path, mode) as f:
    for line in f:
        pass  # do something with line

f.read(size=None)
f.readline()  # 包括'\n'
f.readlines() # 包括'\n'
f.write(bytes_like_or_text_like)
f.writelines(list_of_lines) # 包括'\n'
f.tell()
f.seek(offset[, where])
# SEEK_SET or 0: 绝对位置
# SEEK_CUR or 1: 当前cursor的相对位置
# SEEK_END or 2: 相对文件末尾的位置，通常offset是负数
f.seek(0, os.SEEK_END)
```

mode分别有

- r: 读(default)
- w: 写文件，已存在先清空
- x: 写文件，已存在则报错
- a: append，即在文件末尾添加

should use in conjuction with:

- t: text(default)
- b: binary

text和binary区别是text会decode为unicode，自动处理一写特殊字符如换行。binary不会对读入数据进行任何处理，返回bytes。

## 格式化输出

### str.format

```python
'{0} {1} {name}'.format('arg0', 'arg1', name='value')
'{l[0]} {l[1]}'.format(l=[1, 2])
'{0[0]} {0[1]}'.format([1, 2])
'{d[A]} {d[B]}'.format(d={'A': 65, 'B': 66})
'{0[A]} {0[B]}'.format({'A': 65, 'B': 66})
```

### justify

```python
'{0:15}'.format('leftjust')
'{0:>15}'.format('rightjust')
'{0:>{1}}'.format('dynamic right just', 20)
'dynamic right just'.rjust(20)
```

### zeros and decimal places

```python
'{0:015}'.format(123)
'{0:.4f}'.format(1.23)
'{0:15.4f}'.format(1.23)
```

## 内存模型

全是object，所有变量不管是literal还是name，都是地址

```python
id(0)
a = 0
id(a)
a = -10
a is -10
a = 1
a is 1
a = 255
a is 255
a = 256
a is 256
```

immutable:

- numbers (int, float)
- string
- tuple

mutable:

- list
- dictionary
- set
- objects (custom class)

### assignment, copy and deepcopy

对于immutable，它们都是相同的，所以只用assignment就够了。对于mutable的东西，比如list

assignment

```python
x = ['foo', [4, 2, 3], 10.4]
y = x
```

copy(shallow copy)

```python
x = ['foo', [4, 2, 3], 10.4]
y = x[:]
y = list(x)
y = copy.copy(x)
```

deepcopy(recursive copy)

```python
x = ['foo', [4, 2, 3], 10.4]
y = copy.deepcopy(x)
```

## class 类

### 为什么要使用类

- 面向对象`obj.do()`
- 模块化，命名
- 保持状态
- 复用代码

### 语法

python对类没有太多的硬性要求

```python
class Foo(object):
    count = 0  # class static variable

    def __init__(self, bar):
        self.bar = bar
        self._interval_var = 'internal var'  # actually can be accessed outside
        Foo.count += 1  # do not use self.count

    def f1(self):
        print('normal method needs a self variable', self.bar, self._f1())

    def _f1(self):
        pass

    @classmethod
    def f2(cls):
        print('class method needs a cls variable which is basically the same as Foo', cls.count)

    @staticmethod
    def f3():
        print('static method does not need self or cls variable')
        print('so it cannot access class static variable or instance variable')

    @property
    def bar2(self):
        return self.bar * 2
```

class是定义，object是实例(instance)。self是指向object(instance)本身的指针，每个object的self都不一样。

### 特殊函数

很多很多

```python
class Foo(object):

    def __init__(self, bar):
        self.bar = bar

    def __str__(self):
        return 'Foo[bar={0}]'.format(self.bar)

    def __eq__(self, other):
        return self.bar == other.bar

    def __lt__(self, other):
        return self.bar < other.bar

    def __add__(self, other):
        return Foo(self.bar + other.bar)

    def __sub__(self):
        pass

f = Foo(2)
g = Foo(2)
print(f == g)
print(f + g)
```

## debug

除了`print`我们还有debugger`pdb`！可以让程序在某一行暂停！然后可以查看各种变量！

推荐使用pycharm进行debug，或者用vscode或者atom这样的多功能文本编辑器。日常开发中请抛弃idle！（除非是要求或者电脑所限）

## 迷思

### 为什么避免用全局变量

也不是不能用，比如常量可以，但你们老师给的代码。。。（这课槽点太多了）

1. 所有东西都堆在一起，代码没法读
2. 别的python文件import你的代码时，写在全局的代码会被执行（就是`__name__ == '__main__'`的原因）
3. 同名的局部变量会shadow全局变量
4. 需要全局变量的时候，考虑给函数多加一个参数或者用class（写DFS的时候也求你们别用全局变量，比较具有误导性）

```python
print('name', __name__)
if __name__ == '__main__':
    print('in main')
```

```python
count = 0

def add():
    count += 1
```

### fail fast以及assert模式

错误处理是编程中很重要的一环！尤其python这么高级的语言，coding一时爽，debug火葬场！

业界（各种语言）有两种主流的错误处理方式

1. 通过函数返回值判断：c, go, shell
2. try catch finally：c++, java, python

```python
def dont_pass_me_an_empty_object(obj):
    print(obj[0])

try:
    result = []
    f = open('asd.txt')
    for line in r:
        dont_pass_me_an_empty_object(line)  # may raise IndexError
except:
    raise
finally:
    if f:
        f.close()

result = []
with open('asd.txt') as f:
    for line in r:
        assert line  # fail fast
        dont_pass_me_an_empty_object(line)
```
