---
title: "Binary Tree"
date: 2017-10-24T12:34:12+11:00
---

## 表示

```python
class TreeNode:
    def __init__(self, value, left=None, right=None):
        self.value = value
        self.left = left
        self.right = right

    def __str__(self):
        left_value = self.left.value if self.left else None
        right_value = self.right.value if self.right else None
        return 'TreeNode({0}, left={1}, right={2})'.format(self.value, left_value, right_value)
```

## 遍历 Traversal

### 前序 中序 后序

其实就是非常简单的DFS，树上的操作大多数考虑用DFS

```python
def preorder(node):
    if not node:
        return
    print(node)
    preorder(node.left)
    preorder(node.right)


def postorder(node):
    if not node:
        return
    postorder(node.left)
    postorder(node.right)
    print(node)


def inorder(node):
    if not node:
        return
    inorder(node.left)
    print(node)
    inorder(node.right)
```

### 层序 level order

实际上是BFS，因为先是距离root为0的结点，再是距离root为1的结点...

```python
from collections import deque

def levelorder(root):
    queue = deque()
    queue.append(root)
    while queue:
        node = queue.popleft()
        print(node)
        if node.left:
            queue.append(node.left)
        if node.right:
            queue.append(node.right)
```

## 二叉搜索树

二叉搜索树的每一个结点都满足，左子树中所有结点的值都小于当前结点都值，右子树中所有结点的值都大于当前结点的值。比如

```no-highlight
      5
     / \
    4   8
   /   / \
  2   6   9
 / \       \
1   3       10
```

### 插入

```python
def insert(node, value):
    if not node:
        return TreeNode(value)
    if value < node.value:
        node.left = insert(node.left, value)
    elif value > node.value:
        node.right = insert(node.right, value)
    return node
```

### 验证是否是二叉搜索树

[leetcode指路](https://leetcode.com/problems/validate-binary-search-tree/)

```python
def helper(node, left, right):
    if not node:
        return True
    if not (left < node.value < right):
        return False
    return (helper(node.left, left, node.value)
        and helper(node.right, node.value, right)

def is_valid_bst(root):
    helper(root, float('-inf'), float('inf'))
```

## 其他各种题目

### Path Sum 路径之和

[leetcode指路](https://leetcode.com/problems/path-sum/)

验证二叉树上是否有一条路径的和等于给定sum

```python
def has_path_sum(node, total):
    if not node:
        return total == 0
    total -= node.value
    return has_path_sum(node.left, total) or has_path_sum(node.right, total)
```

### 验证是否同一棵树

[leetcode指路](https://leetcode.com/problems/same-tree/)

```python
def is_same_tree(p, q):
    if not p:
        return not q
    if not q:
        return False
    if p.value == q.value:
        return is_same_tree(p.left, q.left) and is_same_tree(p.right, q.right)
    return False
```

### 验证树本身是否左右对称

[leetcode指路](https://leetcode.com/problems/symmetric-tree/)

对称的

```no-highlight
    1
   / \
  2   2
 / \ / \
3  4 4  3
```

不对称的

```no-highlight
    1
   / \
  2   2
   \   \
   3    3
```

```python
def helper(p, q):
    if not p:
        return not q
    if not q:
        return False
    if p.value == q.value:
        return helper(p.left, q.right) and helper(p.right, q.left)
    return False
```
