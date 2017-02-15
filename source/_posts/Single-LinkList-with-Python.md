---
title: 使用Python实现单链表
toc: true
date: 2016-09-27 21:57:59
tags:
    - Python
    - 数据结构
categories: 
    - Python
---

## 单链表
使用python实现单链表

<!--more--> 

### 代码
```python

class Node(object):

    def __init__(self, data=None, p=None):
        self.data = data
        self.next = p


class SingleLinkList(object):

    def __init__(self, data=None):

        if data == None:
            self.head = None
            self.length = 0

        elif isinstance(data, Node):
            self.head = data
            self.length = 1
        elif isinstance(data, (list, tuple)):
            Len = len(data)
            self.length = Len
            if Len != 0:
                self.head = Node(data[0])
                p = self.head

                for i in range(1, Len):
                    p.next = Node(data[i])
                    p = p.next
            else:
                self.head = None

    def __len__(self):
        return self.length

    def __getitem__(self, key):
        if self.length == 0:
            print('Error: Link is empty.')
            return
        elif key > self.length or key < 0:
            print('Error: Given key is out of range.')
            return
        else:
            return self.get_item(key)

    def __setitem__(self, key, data):
        if key > self.length + 1 or key < 0:
            print('Error: Give key is out of range.')
            return
        else:
            return self.set_item(key, data)

    def __delitem__(self, key):
        if self.length == 0:
            print('Error: Linklist is empty')
            return
        elif key < 0 or key > self.length - 1:
            print('Error: Give key is out of range.')
            return
        else:
            return self.del_item(key)

    def __str__(self):
        p = self.head
        if self.length > 0:
            L = str(p.data)
            p = p.next
            while p != None:
                L += ' -> ' + str(p.data)
                p = p.next
        else:
            L = 'Empty LinkList'
        return L

    def get_item(self, key):

        p = self.head
        j = 0
        while p != None and j < key:
            p = p.next
            j += 1
        if p == None:
            print('Error')
            return None
        return p.data

    def set_item(self, key, data):
        node = Node(data)
        j = 0
        p = self.head
        while p != None and j < key - 1:
            p = p.next
            j += 1
        if p == None or j > key - 1:
            print('Error')
            return

        node.next = p.next
        p.next = node
        self.length += 1

    def del_item(self, key):
        j = 0
        p = self.head

        while j < key - 1 and p.next != None:
            p = p.next
            j += 1
        if p.next == None:
            p = None
            self.length -= 1
            return

        q = p.next
        p.next = q.next
        self.length -= 1
        return q.data

    def isEmpty(self):
        return self.head == None

    def append(self, data):
        temp = Node(data)
        if self.isEmpty():
            self.head = temp
        else:
            p = self.head
            while p.next != None:
                p = p.next

            p.next = Node(data)
        self.length += 1

    def index(self, data):
        p = self.head
        i = 0
        while p != None:
            if p.data == data:
                return i
            else:
                p = p.next
                i += 1
        else:
            return -1

    def clear(self):
        self.head = None
        self.length = 0


if __name__ == '__main__':
    l = SingleLinkList([1, 2, 3, 4, 5])
    print(l)
    l[3] = 22

    del l[4]
    print(l)
    print(l.index(5))

```
### 输出
```bash
1 -> 2 -> 3 -> 4 -> 5
1 -> 2 -> 3 -> 22 -> 5
4
```
