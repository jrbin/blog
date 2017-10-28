---
title: "Linked List"
date: 2017-10-20T18:36:53+11:00
---

## 表示

```python
class ListNode:
    def __init__(self, value, next=None):
        self.value = value
        self.next = next

def print_list(head):
    p = head
    while p:
        print(p.value, end=' ')
        p = p.next
    print()

def delete_node(head, value):
    previous = None
    p = head
    while p and p.value != value:
        previous = p
        p = p.next
    if p:
        if previous:
            previous.next = p.next
        else:
            head = p.next
        p.next = None
    return head

def add_node(head, value):
    previous = None
    p = head
    while p and p.value <= value:
        previous = p
        p = p.next
    n = ListNode(value)
    if previous:
        previous.next = n
    else:
        head = n
    n.next = p
    return head

def merge_list(h1, h2):
    if not h1:
        return h2
    if not h2:
        return h1

    head = h1 if h1.value < h2.value else h2
    p1 = h1
    p2 = h2
    while p1 and p2:
        if p1.value >= p2.value:
            temp = p1
            p1 = p2
            p2 = temp
        if not p1.next or p1.next.value >= p2.value:
            temp = p1.next
            p1.next = p2
            p1 = temp
        else:
            p1 = p1.next
    return head

def main():
    head = None
    for i in range(1, 4)[::-1]:
        head = ListNode(i, head)

    print_list(head)
    head = delete_node(head, 1)
    print_list(head)
    head = add_node(head, 1)
    head = add_node(head, 4)
    print_list(head)

    h1 = None
    for i in [1, 4, 5][::-1]:
        h1 = ListNode(i, h1)

    h2 = None
    for i in [2, 3, 6, 7][::-1]:
        h2 = ListNode(i, h2)

    print_list(h1)
    print_list(h2)
    head = merge_list(h1, h2)
    print_list(head)
```
