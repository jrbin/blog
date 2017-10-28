---
title: "Breadth First Search"
date: 2017-10-24T19:25:25+11:00
---

## 框架

BFS的大意是从某一点开始，搜索距离为1的所有点，再搜索距离为2的点...

所以，BFS通常可以用来找最短路径，而DFS不行。最基本的BFS是树的层序（levelorder）遍历。

```python
def bfs(initial_state):
    queue = deque()
    queue.append(initial_state)
    while queue:
        state = queue.popleft()
        # do something with current state
        for new_state in explore_neighbours(state):
            queue.push(state)
```

## 题目

### 树的最小深度

[leetcode指路](https://leetcode.com/problems/minimum-depth-of-binary-tree/)

这题用DFS也可以做，而且代码很简单。但DFS的缺点是要搜索树的所有点，万一树是十分不平衡的情况就会比较差。考虑到是最小深度，可以用BFS，而且代码和levelorder很像。

```python
def min_depth(root):
    if not root:
        return 0
    queue = deque()
    queue.append(root)
    end = root
    depth = 1
    while queue:
        node = queue.popleft()
        if not node.left and not node.right:
            break
        if node.left:
            queue.append(node.left)
        if node.right:
            queue.append(node.right)
        if node == end:
            depth += 1
            end = queue[-1]
    return depth
```

### 迷宫最短路径长度

从左上角开始，`0`是可以走的区域，`1`是墙，找到`2`的最短路径长度

```
000010
001110
000100
110002
```

上面这个例子答案应该是`8`

```python
from collections import deque

DX = [1, 0, -1, 0]
DY = [0, 1, 0, -1]

def bfs(matrix, x, y):
    queue = deque()
    queue.append((x, y, 0))
    visited = set()
    while queue:
        x, y, length = queue.popleft()
        if matrix[x][y] == 2:
            return length
        visited.add((x, y))
        for i in range(4):
            nx = x + DX[i]
            ny = y + DY[i]
            if 0 <= nx < len(matrix) and 0 <= ny < len(matrix[0]) and matrix[nx][ny] != 1 and (nx, ny) not in visited:
                queue.append((nx, ny, length + 1))
```

### Lab9 Q2 word ladder

这题是最短路径，也是典型的BFS。

函数声明方面我参照的是[leetcode题目](https://leetcode.com/problems/word-ladder-ii/)而不是老师lab中给的。老师在lab中说的可以先建立一个相邻字典（即字典的key是word，value是和这个word只差一个字符的其他所有words），但实际上这个操作很花时间。所以这里采用对于一个词，首先把它第一位的字符改成不同的字符，然后第二位改成不同的字符....看它是否在字典里这样的操作。（这个的实现在函数`neighbours()`中）

这题有两个难点，一个是要判断BFS的层数（深度），一个是要回溯路径。当然我们有很多种方法解决这两个问题。见下面两个不同版本的实现。

`findLadders3()`是一个优化过的BFS，它是一层一层来的，并且把每一层的词的前一个词都存在了parent这个字典里。所以上面两个问题就一次性解决了。

`findLadders4()`是一个比较朴素的BFS实现，它判断层数是在每一层的结尾加上一个magic element`None`。它的回溯是通过在queue中除了word本身也塞入一个上一个word的引用。

注意leetcode上时限很严，`findLadders4()`超时，`findLadders3()`勉强过。要想更快，要用双向BFS的版本，这里就不讲了。

```python
from collections import deque, defaultdict

def neighbours(word):
    chars = list(word)
    for i, char in enumerate(chars):
        for new_char in 'abcdefghijklmnopqrstuvwxyz':
            if new_char != char:
                chars[i] = new_char
                yield ''.join(chars)
        chars[i] = char

def findLadders3(beginWord, endWord, wordList):
    wordSet = set(wordList)
    level = {beginWord}
    parents = defaultdict(set)
    while level and endWord not in parents:
        next_level = defaultdict(set)
        for word in level:
            for childWord in neighbours(word):
                if childWord in wordSet and childWord not in parents:
                    next_level[childWord].add(word)
        level = next_level
        parents.update(next_level)

    result = [[endWord]]
    while result and result[0][0] != beginWord:
        result = [[p] + row for row in result for p in parents[row[0]]]
    return result

def findLadders4(beginWord, endWord, wordList):
    words = set(wordList)
    queue = deque()
    queue.append((beginWord, None))
    queue.append(None)
    visited = set()
    met = False
    result = []
    while queue:
        pair = queue.popleft()
        if pair is None:
            if met or not queue:
                break
            else:
                queue.append(None)
                continue
        word, prev = pair
        visited.add(word)
        for new_word in neighbours(word):
            if new_word == endWord:
                if not met:
                    met = True
                result.append(pair)
            if new_word in words and new_word not in visited:
                queue.append((new_word, pair))

    for i, pair in enumerate(result):
        result[i] = [endWord]
        while pair:
            result[i].append(pair[0])
            pair = pair[1]
        result[i].reverse()
    return result
```
