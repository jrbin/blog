---
title: Template Friend Function
date: 2017-09-22T10:58:50+10:00
tags:
  - c/c++
---

c++中，如果template class中有friend函数，比如常见的重载operator<<是friend，可能就会报一个尴尬的错误

```c++
template <typename T>
class btree {
public:
  // see https://stackoverflow.com/q/4660123
  friend std::ostream& operator<<(std::ostream& out, const btree<T>& tree);
};
```

```shell
error: friend declaration 'std::ostream& operator<<(std::ostream&, const btree<T>&)' declares a non-template function [-Werror=non-template-friend]
   friend std::ostream& operator<<(std::ostream& out, const btree<T>& tree);
                                                                          ^
```

这个错误我试过在gcc4.9和gcc6.3上都会报，在clang上不会报。

其实用脚趾头猜猜就能想到因为friend本身不属于这个class，它就无法利用这个class的template参数T。但其实一般人写成这样应该觉得没什么问题，别的程序员也能读懂。

这里吐槽一下，通过这个我们发现c++就是一个不符合程序员逻辑的语言，gcc是一个没有逻辑的编译系统。编译器上的问题交给c++语法解决，导致c++不是一个通过人类逻辑设计的语言，而是一个用编译器逻辑设计的语言。

说一下解决方法。[这个问答](https://stackoverflow.com/q/4660123)给我们提供了两种不同的思路。

## 方法1

直接把friend也声明成一个template函数，注意friend的template是不依赖于class的template的。而且typename不能写成T否则会shadow T。

```c++
template <typename T>
class btree {
public:
  template <typename U>
  friend std::ostream& operator<<(std::ostream& out, const btree<U>& tree);
};
```

好好的函数非要写成这样，扩大它的接受范围，我又能说啥呢。

## 方法2

inline的实现，就是把实现直接写声明那块。

```c++
template <typename T>
class btree {
public:
  friend std::ostream& operator<<(std::ostream& out, const btree<T>& tree) {
    out << "yeah";
    return out;
  }
};
```

这也不是特别好，因为声明与定义分离是c/c++的重要原则。（都template说什么声明定义分离，尴尬）

## 方法3

课上提到的，也可以参考[这个](http://en.cppreference.com/w/cpp/language/friend#Template_friend_operators)

也就是说，首先声明friend是一个template，然后在template class中用参数T具体化它。其实跟方法1差不多，但是每个class中这个operator<<都是跟T绑定的。而方法1中的operator<<可以接受btree\<U\>, btree\<V\>, whatever.

```c++
template <typename T> class btree;
template <typename T> std::ostream& operator<<(std::ostream& os, const btree<T>& tree);

template <typename T>
class btree {
public:
  friend std::ostream& operator<< <T> (std::ostream& out, const btree<T>& tree);
};
```
