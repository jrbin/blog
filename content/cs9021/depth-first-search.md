---
title: "Depth First Search"
date: 2017-10-20T19:50:33+11:00
tags:
    - "depth first search"
---

## 框架

```python
def helper(state):
    if stop condition is satisfied:
        return
    do some meaningful thing in current step
    for new_state in explore_neighbours(state):
        helper(new_state)

def wrapper():
    initialize state
    helper(state)
```

## 矩阵中连通区域数量

### 题目

题目来自[leetcode](https://leetcode.com/problems/number-of-islands/description/)

计算矩阵中连通区域数量。一个矩阵的连通区域是用多个`1`组成。一个连通区域从自己的任意一个`1`可以达到自己区域中另外一个`1`，但它的外围都被`0`所包围而不能达到矩阵中其他的`1`（即其他的连通区域）。我们定义的连通是上下左右四个方向（即对角线不算）。

__Example 1:__

```
11110
11010
11000
00000
```

Answer: 1

__Example 2:__

```
11000
11000
00100
00011
```

Answer: 3

### 解法

对于矩阵中每个点，如果是`1`，就从它开始找包含它但连通区域并把连通区域上所有点置为0（DFS）（这样就不会重复计算连通区域），并且count加一。

```python
DX = [1, 0, -1, 0]
DY = [0, 1, 0, -1]

def dfs(matrix, i, j):
    # 如果四周都是0，return
    matrix[i][j] = 0
    for k in range(4):
        ni = i + DX[k]
        nj = j + DY[k]
        if 0 <= ni < len(matrix) and 0 <= nj < len(matrix[0]) and matrix[ni][nj] == 1:
            dfs(matrix, ni, nj)

def num_components(matrix):
    count = 0
    for i in range(len(matrix)):
        for j in range(len(matrix[0])):
            if matrix[i][j] == 1:
                dfs(matrix, i, j)
                count += 1
    return count
```

## DFS走迷宫

### 题目

给定一个矩阵，`0`是可以走的地方，`1`是障碍。用DFS找出一条从左上角走到右下角的路径。开始方向是向下，总是试图保持一个方向。如果不行，向左拐。（和之前一个quiz差不多）把最后路径标上`9`，然后把矩阵打印出来。

__Example 1:__

```
00010
11010
11000
00000
```

Answer:

```
99910
11910
11900
00999
```

Answer: 3

### 解法

```python
DX = [1, 0, -1, 0]
DY = [0, 1, 0, -1]

def dfs(matrix, x, y, d, stack):
    matrix[x][y] = 9
    stack.append((x, y))
    print(str(stack)[7:-2])
    if x == len(matrix) - 1 and y == len(matrix[0]) - 1:
        return True
    for _ in range(4):
        nx = x + DX[d]
        ny = y + DY[d]
        if 0 <= nx < len(matrix) and 0 <= ny < len(matrix[0]) and matrix[nx][ny] == 0:
            if dfs(matrix, nx, ny, d, stack):
                return True
        d = (d + 1) % 4
    matrix[x][y] = 0
    stack.pop()
    return False

def find_path(matrix):
    stack = deque()
    if dfs(matrix, 0, 0, 0, stack):
        print('We have found a path!')
    else:
        print('No possible path.')
```

## Lab10 Q2

### 题目

给定一个数组a，在数字中间加上括号和减号，使得算数结果等于给定值n（可能会有多个结果，每行输出一个可能结果）

__Example 1:__

`(1, 2, 3, 4, 5), 1`

Answer:

```
1 - ((2 - 3) - (4 - 5))
(1 - ((2 - 3) - 4)) - 5
```

__Example 2:__

`(1, 3, 2, 5, 11, 9, 10, 8, 4, 7, 6), 40`

Answer:

```
1 - ((((3 - 2) - 5) - 11) - (9 - ((((10 - 8) - 4) - 7) - 6)))
1 - ((((((3 - 2) - 5) - 11) - 9) - 10) - (8 - (4 - (7 - 6))))
1 - ((((((3 - 2) - 5) - 11) - 9) - 10) - ((8 - (4 - 7)) - 6))
1 - (((((((3 - 2) - 5) - 11) - 9) - 10) - (8 - 4)) - (7 - 6))
1 - (((((3 - 2) - 5) - 11) - (9 - (((10 - 8) - 4) - 7))) - 6)
1 - ((((((3 - 2) - 5) - 11) - (9 - ((10 - 8) - 4))) - 7) - 6)
1 - (((((((3 - 2) - 5) - 11) - (9 - (10 - 8))) - 4) - 7) - 6)
1 - ((((((((3 - 2) - 5) - 11) - (9 - 10)) - 8) - 4) - 7) - 6)
(1 - 3) - ((((2 - 5) - 11) - 9) - (10 - (((8 - 4) - 7) - 6)))
(1 - 3) - (((((2 - 5) - 11) - 9) - (10 - ((8 - 4) - 7))) - 6)
(1 - 3) - ((((((2 - 5) - 11) - 9) - (10 - (8 - 4))) - 7) - 6)
(1 - 3) - (((((((2 - 5) - 11) - 9) - (10 - 8)) - 4) - 7) - 6)
(1 - ((((3 - 2) - 5) - 11) - 9)) - ((((10 - 8) - 4) - 7) - 6)
((1 - 3) - ((((2 - 5) - 11) - 9) - 10)) - (((8 - 4) - 7) - 6)
(1 - ((((((3 - 2) - 5) - 11) - 9) - 10) - 8)) - (4 - (7 - 6))
(1 - ((((((3 - 2) - 5) - 11) - 9) - 10) - (8 - (4 - 7)))) - 6
(1 - (((((((3 - 2) - 5) - 11) - 9) - 10) - (8 - 4)) - 7)) - 6
((1 - ((((((3 - 2) - 5) - 11) - 9) - 10) - 8)) - (4 - 7)) - 6
```

### 解法

```python
def helper(array):
    assert array
    if len(array) == 1:
        return [(array[0], str(array[0]))]
    result = []
    for i in range(1, len(array)):
        for lhs in helper(array[:i]):
            for rhs in helper(array[i:]):
                result.append((lhs[0] - rhs[0], '({0} - {1})'.format(lhs[1], rhs[1])))
    return result

def subtractions(array, total):
    print('-------------')
    print('find answer for', array, total)
    for line in helper(array):
        if line[0] == total:
            print(line[1][1:-1])
    print()
```
