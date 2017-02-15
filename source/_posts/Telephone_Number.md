---
title: 电话号码分身
toc: true
date: 2016-09-24 21:44:19
tags:
    - Python
categories:
    - Python
---
继MIUI8推出手机分身功能之后，MIUI9计划推出一个电话号码分身的功能：首先将电话号码中的每个数字加上8取个位，然后使用对应的大写字母代替`("ZERO", "ONE", "TWO", "THREE", "FOUR", "FIVE", "SIX", "SEVEN", "EIGHT", "NINE")`，然后随机打乱这些字母，所生成的字符串即为电话号码对应的分身。



1. 输入
第一行是一个整数T（1<=T<=100）表示测试样例数；接下来T行，每行给定一个分身后的电话号码的分身（长度在3到10000之间）

2. 输出
输出T行，分别对应输入中每行字符串对应的分身的最小电话号码（允许前导0）.

<!--more-->

Python 代码为:

```python
from copy import deepcopy
words = ('ZERO',  'ONE', 'TWO', 'THREE', 'FOUR',
         'FIVE', 'SIX', 'SEVEN', 'EIGHT', 'NINE')


patterns = [{'0': 'Z', '2': 'W', '4': 'U', '6': 'X', '8': 'G'}, {
    '1': 'O', '3': 'T', '5': 'F', '7': 'S'}, {'9': 'I'}]


def count_number(lists, pattern):
    new_count_dict = {}
    for num, patt in pattern.items():
        new_count_dict[num] = lists.count(patt)
    return new_count_dict


def del_number(lists, count_dict):
    for num, count in count_dict.items():
        for i in range(count):
            for cc in words[int(num)]:
                lists.remove(cc)
    return lists


def reform_number(min_number, count_dict):
    for num, count in count_dict.items():
        real_num = int(num) - 8 if int(num) >= 8 else 8 - int(num)
        min_number.append(str(real_num) * count)
    print(min_number)

    return ''.join(sorted(min_number))

if __name__ == '__main__':
    num = '9876543210'

    num_string = []
    for c in num:
        num_string.extend(words[int(c)])
        num_string.sort()
    s = deepcopy(num_string)
    print('输入的数字是:', num)

    count_dict = {}
    min_number = []

    for i in range(3):
        new_count_dict = count_number(s, patterns[i])
        count_dict.update(new_count_dict)
        s = del_number(s, new_count_dict)

    print('统计数字次数为:\n', sorted(count_dict.items()))
    min_number = reform_number(min_number, count_dict)
    print('最小的号码为:', min_number)

```
输出为:
```
输入的数字是: 9876543210
统计数字次数为:
 [('0', 1), ('1', 1), ('2', 1), ('3', 1), ('4', 1), ('5', 1), ('6', 1), ('7', 1)
, ('8', 1), ('9', 1)]
['5', '3', '8', '1', '0', '6', '1', '4', '2', '7']
最小的号码为: 0112345678
```
