---
title: "Lab6 - Lab10"
date: 2017-10-24T16:37:30+11:00
---

## Lab6 Q1

可用DFS生成所有组合情况（老师给的答案，但他给的没有剪枝）

```python
def dfs(digits, target):
    if target == 0:
        return 1
    if not digits or target < 0:
        return 0
    return dfs(digits[1:], target - int(digits[0])) + dfs(digits[1:], target)
```

或者使用`itertools.combination`，这种方法可能没上种方法好因为没法剪枝。

```python
def comb(digits, target):
    count = 0
    for i in range(1, len(digits) + 1):
        for item in combinations(digits, i):
            if sum([int(i) for i in item]) == target:
                count += 1
    return count
```

## Lab6 Q2

老师给了一个recursive的解法也是生成所有可能的情况的。复杂度稍高。

```python
def dfs(a, b, c):
    if not b:
        return a == c
    if not c:
        return a == b
    if not a:
        return False
    if a[0] == b[0] and dfs(a[1:], b[1:], c):
        return True
    if a[0] == c[0] and dfs(a[1:], b, c[1:]):
        return True
    return False
```

这边给一个清晰明了的`O(n)`的iterative解法，唯一缺点是空间也是`O(n)`

```python
def merge_strings(a, b, c):
    print(a)
    print(b)
    print(c)
    longest = 0
    if len(b) > len(a) and len(b) > len(c):
        t = b
        b = a
        a = t
        longest = 1
    if len(c) > len(a) and len(c) > len(b):
        t = c
        c = a
        a = t
        longest = 2
    if len(a) != len(b) + len(c):
        print('Impossible')
        return
    bi = 0
    used = set()
    for ai, char in enumerate(a):
        if char == b[bi]:
            bi += 1
            used.add(ai)
            if bi == len(b):
                break
    if bi < len(b) or c != ''.join([a[i] for i in range(len(a)) if i not in used]):
        print('Impossible')
        return
    print('The {0} string can be obtained by merging the other two'.format(
        ['first', 'second', 'third'][longest]))
```

## Lab6 Q3

老师的写法比较暴力，二重循环搞定。考试的时候如果想了5分钟还没有想到特别好的方法就怎么粗暴怎么解决。应该不会特别卡时间。

这边提供一个更好的方法。有动态规划（dynamic programming）的思路。（所谓动态规划，就是在解决问题的某一步中利用前一步或者前面所有步骤的结果）

考虑用一个长度为26的数组存放到当前位置以字母`c`结尾的最长连续字母的长度。

```python
def longest_sequence(letters):
    dp = [0] * 26
    for i, char in enumerate(letters):
        char_id = ord(char.lower()) - ord('a')
        if char_id == 0:
            dp[0] = 1
        else:
            dp[char_id] = max(dp[char_id], dp[char_id - 1] + 1)
    max_length = max(dp)
    index_end = dp.index(max_length)
    for i in range(index_end - max_length, index_end):
        print(chr(98 + i), end='')
    print()
```

但题目中有个要求，就是有相等长度的，取开始字符里开头最近的那个。所以我们考虑在`dp`数组中多存一个开始位置的信息。

```python
def longest_sequence(letters):
    dp = [(0, -1)] * 26
    for i, char in enumerate(letters):
        char_id = ord(char.lower()) - ord('a')
        if char_id == 0:
            if dp[0][1] < 0:
                dp[char_id] = (1, i)
        else:
            if dp[char_id - 1][0] + 1 > dp[char_id][0]:
                if dp[char_id - 1][1] < 0:
                    index_begin = i
                else:
                    index_begin = dp[char_id - 1][1]
                dp[char_id] = (dp[char_id - 1][0] + 1, index_begin)
    max_pair = max(dp, key=lambda x: (x[0], -x[1]))
    index_end = dp.index(max_pair)
    for i in range(index_end - max_pair[0], index_end):
        print(chr(98 + i), end='')
    print()
```

这样就完全正确了

## Lab7 Q1

思路是greedy，即每一步都取局部最优。知道这种思路就好了，实现很简单。（需要注意的是有些问题每一步去局部最优并不能达到全局最优）

## Lab7 Q3

不讲

## Lab8 Q1

没有任何算法的题，题比较长，仔细读就好了。

```python
def encode(num):
    if num == 1:
        return '11'
    if num == 2:
        return '011'
    fibs = [1, 2]
    while not fibs[-2] <= num < fibs[-1]:
        fibs.append(fibs[-2] + fibs[-1])
    fibs.pop()
    result = []
    for i in range(0, len(fibs))[::-1]:
        if num >= fibs[i]:
            num -= fibs[i]
            result.append('1')
        else:
            result.append('0')
    return ''.join(result[::-1]) + '1'


def decode(string):
    if not string or len(string) == 1:
        return 0
    if not (string[-1] == '1' and string[-2] == '1'):
        return 0
    fibs = [1, 2]
    string = string[:-1]
    while len(fibs) < len(string):
        fibs.append(fibs[-2] + fibs[-1])
    result = 0
    prev = None
    for i, char in enumerate(string):
        if char == '1':
            if prev == '1':
                return 0
            result += fibs[i]
        prev = char
    return result
```

## Lab8 Q2 Q3

都是关于linked list链表的，具体看[Linked List](/cs9021/linked-list/)

## Lab9 Q1

上周讲了，见[Stack](/cs9021/stack/)

## Lab9 Q2

见[BFS](/cs9021/breadth-first-search/)

## Lab10 Q1

在Lab9 Q1的基础上进行一些小的修改就行。

比如Lab9 Q1每次往stack里push数字的时候改成push TreeNode。每次处理括号里的算术的时候改为把运算符左右的TreeNode放在新TreeNode的左右孩子上，再把新TreeNode push进stack就行。

```python
from collections import deque

MATCH = {
    ')': '(',
    ']': '[',
    '}': '{'
}


def calculate(stack, c):
    ops = []
    while stack:
        top = stack.pop()
        if top == MATCH[c]:
            break
        ops.append(top)
    else:
        assert False
    assert len(ops) == 3
    node = TreeNode(ops[1], ops[2], ops[0])
    stack.append(node)


def parse_tree(s):
    print(s)
    stack = deque()
    num = 0
    num_pushed = True
    for c in s:
        if '0' <= c <= '9':
            num = 10 * num + int(c)
            num_pushed = False
        else:
            if not num_pushed:
                stack.append(TreeNode(num))
                num = 0
                num_pushed = True
            if c in '([{+-*/':
                stack.append(c)
            elif c in ')]}':
                calculate(stack, c)
            else:
                pass
    if stack:
        if len(stack) == 1:
            return stack.pop()
        assert False
    return TreeNode(num)
```

## Lab10 Q2

上周讲了，见[DFS](/cs9021/depth-first-search/#lab10-q2)
