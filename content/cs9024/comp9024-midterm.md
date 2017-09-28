---
title: COMP9024 Midterm
date: 2017-09-14T13:27:51.000Z
tags:
  - comp9024
---

## Pointers

![typical memory arrangement](/img/mem.jpg)

### Pass by value

```c
void swap(int a, int b) {
    int t = a;
    a = b;
    b = t;
}

void swap(int *a, int *b) {
    int t = *a;
    *a = *b;
    *b = t;
}

typedef struct {
    int x;
    int y;
} Point;

void init_point(Point *dst) {
    dst->x = 1;
    dst->y = 2;
}

Point *p = (Point *) malloc(sizeof(Point));
init_point(p);
free(p);
```

### Exercise

```c
int data[12] = {5, 3, 6, 2, 7, 4, 9, 1, 8};
// data == 0x00010000
```

Expression            | Result
--------------------- | ----------
data + 4              | 0x00010010
*data + 4             | 9
*(data + 4)           | 7
data[4]               | 7
\*(data + *(data + 3)) | 6
data[data[2]]         | 9

```c
typedef struct {
   int   studentID;
   int   age;
   char  gender;
   float WAM;
} PersonT;

PersonT per1;
PersonT per2;
PersonT *ptr;

ptr = &per1;
per1.studentID = 3141592;
ptr->gender = 'M';         // per1.gender = 'M'
ptr = &per2;
ptr->studentID = 2718281;  // per2.studentID = 2718281
ptr->gender = 'F';         // per2.gender = 'F'
per1.age = 25;
per2.age = 24;
ptr = &per1;
per2.WAM = 86.0;
ptr->WAM = 72.625;         // per1.WAM = 72.625
```

## Memory management

### Return a pointer

```c
// wrong
int *make_array() {
   int arr[] = {1, 2, 3, 4, 5};
   return arr;
}

int *array = make_array();
```

问题在于`make_array()`内部的arr是函数的局部变量，它占有的内存（栈内存）在函数退出后将被清空。array会指向一个可能会被修改的区域。尽管打印array可能能得到正确的值。

正确一点的做法

```c
int *make_array() {
    int n_arr = 5;
    int *arr = (int *) malloc(n_arr * sizeof(int));
    for (int i = 0; i < n_arr; i++) {
        arr[i] = i + 1;
    }
    return arr;
}
```

此时arr是堆内存，函数返回以后不会被释放，所以没有问题。堆内存在C语言中要程序员自己管理，操作系统并不知道什么时候去释放它，所以记得malloc之后一定要有free。（但程序退出后会被操作系统释放，所以课堂上写的这种小程序就算忘了free也并不会有什么问题）

### Change a pointer

```c
// wrong
void func(int *a) {
   a = malloc(sizeof(int));
}

int main(void) {
   int *p = NULL; // (void *) 0x0
   func(p);
   // p == NULL
   *p = 6;
   printf("%d\n",*p);
   free(p);
   return 0;
}
```

问题在于在调用函数func之后，p指向的地址仍然是NULL，因为函数无法改变a本身的值，只能改变a指向的值。要改变一个指针，要传`int **`

### Unsigned and signed

> **WRONG!** : The placeholder %lld (instead of %d) can be used to print an unsigned long long int.

```c
#include <limits.h>
unsigned int a = UINT_MAX;
printf("%d\n", a);   // WRONG
printf("%u\n", a);   // correct
unsigned long long b = ULLONG_MAX;
printf("%lld\n", b); // WRONG
printf("%llu\n", b); // correct
```

```c
// WRONG
for (unsigned i = 100; i >= 0; i--) {
    // do something
}
// correct
for (unsigned i = 100; i-- > 0; ) {
    // do something
}
for (int i = 100; i >= 0; i--) {
    // do something
}
```

任何情况下不建议使用`unsigned`，除非在做位操作。（比如`&, |, ^`）unsigned也不代表非负整数。除非你真的觉得需要比signed多出的那一点空间。

```
unsigned + int -> unsigned
unsigned * int -> unsigned
unsigned - unsigned -> unsigned
```

## Linked list

```c
typedef struct ListNode {
    int value;
    struct ListNode *next;
} ListNode;

// ADT
typedef ListNode *List;

List ListInit();
void ListDestroy(List l);
void ListAdd(List l, int value);
int ListGet(List l, int index);
int ListRemove(List l, int index);
int ListEmpty();
List ListConcat(List l1, List l2);
```

![list](/img/list.gif)

## Analysis of algorithms

### Big-Oh notation

a. \(\sum_{i=1}^n i^2\)

1. \(O(n^2)\)
2. \(O(n^3)\)
3. \(O(n^4)\)
4. \(O(n^3 \log{n})\)

b. \(\sum_{i=1}^n \log{i}\)

1. \(O(\log{n})\)
2. \(O(n)\)
3. \(O(n \log{n})\)
4. \(O(n^2)\)

c. Show that if \(p(n)\) is any polynomial in n, then \(\log{p(n)}\) is \(O(\log{n})\)

\(p(n) < C \cdot n^k\)

### Design find_primes

Design an algorithm to output primes in a range [a, b] (2 <= a <= b), and show its time complexity.

```c
int is_prime(int n) {
    for (int i = 2; i <= n / 2; i++) {
        if (n % i == 0) {
            return 1;
        }
    }
    return 0;
}

void find_primes(int a, int b) {
    for (int i = a; i <= b; i++) {
        if (is_prime(i)) {
            printf("%d\n", i);
        }
    }
}
```

\(O(n^2)\)

### Design is_palindrome

```c
int is_palindrome(char *s) {
    int len = strlen(s);
    for (int i = 0; i < len / 2; i++) {
        if (s[i] != s[len - 1 - i]) {
            return 0;
        }
    }
    return 1;
}
```

\(O(n)\)

### Design algorithm polynomial

Let \(p(x)=\sum_0^n a_i x^i\), given coefficient array a and x, design a function calculate p(x)

```c
int polynomial(int *arr, int arr_size, int x) {
    int result = 0;
    for (int i = arr_size - 1; i >= 0; i--) {
        result = x * result + arr[i];
    }
    return result;
}
```

\(O(n)\)

### Design algorithm merge_list

不要求会写C代码，要求知道思路和能写伪代码

```c
function merge_list(head_a, head_b) {
    while (not end(head_a) and not end(head_b)) {
        if (head_a < head_b) {
            append(result_list, head_a)
            next(head_a)
        } else {
            append(result_list, head_b)
            next(head_b)
        }
    }
    while (not end(head_a)) {
        append(result_list, head_a)
        next(head_a)
    }
    while (not end(head_b)) {
        append(result_list, head_b)
        next(head_b)
    }
    return result_list
}
```

\(O(n+m)\)

## Graph

### Terminologies

![graph terminologies](/img/graph2.png)

- connected graph（连通图）：从每个顶点可以走到任何其他顶点，如果不是连通图，则它至少有2个connected graph component（连通图分量）
- complete graph：每个顶点和所有另外的顶点都有边相连
- spanning tree （生成树）： 没有cycle的connected (sub)graph，并且要包括所有顶点
- clique: complete subgraph

### Representations

#### ADT

```c
typedef GraphRep *Graph;

typedef int Vertex;

Graph GraphInit(int n_vertices);
void GraphDestroy(Graph g);
void GraphInsertEdge(Graph g, Vertex src, Vertex dst, int weight);
void GraphRemoveEdge(Graph g, Vertex src, Vertex dst);
int GraphAdjacent(Graph g, Vertex src, Vertex dst);
```

#### Array of edges

![array of edges](/img/graph-array-edges.png)

```c
// Graph = array of Edge(src, dst, weight)

typedef struct Edge {
    int src;
    int dst;
    int weight;
} Edge;

typedef struct GraphRep {
    Edge* edges;
    int n_vertices;
    int n_edges;
} GraphRep;
```

Space complexity: O(E)

Time complexity (assume array implementation):

- init: O(1)
- insert edge: O(E)
- delete edge: O(E)
- find edge: O(E) (O(logE) if array is kept sorted)

相对少用，因为各种操作都比较慢，但在特定的算法比如Bellman Ford中非常方便。

#### Adjacency Matrix

![adjacency matrix](/img/adjmatrix.png)

```c
typedef struct GraphRep {
    int **matrix;
    int n_vertices;
    int n_edges;
} GraphRep;
```

Space complextixy: \(O(V^2)\)

Time complexity:

- init: \(O(V^2)\)
- insert edge: O(1)
- delete edge: O(1)
- find edge: O(1)

不好的地方就是space inefficient，因为大多数图都是sparse的，也就是说边的数量远小于上限，矩阵里就会有很多不用的空间。

#### Adjacency List

![adjacency list](/img/adjlist.jpg)

```c
// see linked list section about its definition
typedef struct GraphRep {
    List *matrix;
    int n_vertices;
    int n_edges;
} GraphRep;
```

Space complextixy: O(V+E) (lecture slide上O(E)不准确)

Time complexity:

- init: O(V)
- insert edge: O(1) (O(E) if list is sorted)
- delete edge: O(E)
- find edge: O(E)

### Comparison

operation     | array of edges | adjacency matrix | adjacency list
------------- | -------------- | ---------------- | --------------
space         | E              | \(V^2\)          | V+E
init          | 1              | \(V^2\)          | V
insert edge   | E              | 1                | 1
remove edge   | E              | 1                | 1
remove node   | E              | V                | 1
hasPath(x, y) | ElogV?         | \(V^2\)          | V+E
copy graph    | E              | \(V^2\)          | V+E
destroy graph | 1              | V                | V+E
