---
title: "Binary Heap"
date: 2017-10-24T16:23:07+11:00
---

## 表示

binary heap二叉堆（priority queue一般就是大顶二叉堆）是一种完全二叉树。而且每个结点都大于它的子节点（小顶则是小于）。

![binary heap](/img/heap.png)

由于是一个完全二叉树，所以一般heap可以用一个数组（python list存）。其中第0个元素不存东西，树的结点依层序遍历的顺序存在数组中。

这样，我们可以轻易的得到一个元素的parent, left\_child, right\_child

```
parent_index = index // 2
left_child_index = index * 2
right_child_index = index * 2 + 1
```

## 操作

### 插入

每次插入数组的最后，向上冒泡以满足heap的定义

### 删除堆顶

删除堆顶后将数组最后元素替换到堆顶，向下冒泡以满足heap的定义

### 删除堆中任一元素

删除该位置元素后将数组最后元素替换到该位置，依据情况向上或者向下冒泡
