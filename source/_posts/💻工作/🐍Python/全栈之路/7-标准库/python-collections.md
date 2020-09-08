---
title: Python 标准库系列之 collections 模块
toc: true
tags:
  - 编码
  - 面向对象
top: 16
categories:
  - "\U0001F4BB工作"
  - "\U0001F40DPython"
  - 全栈之路
  - 7-标准库
date: 2020-05-23 18:21:46
---
## 引言

Python 作为一个“内置电池”的编程语言，标准库里面拥有非常多好用的模块。比如今天想给大家介绍的 collections 就是一个非常好的例子。

> 官方文档：https://docs.python.org/3/library/collections.html

This module implements specialized container datatypes providing alternatives to Python’s general purpose built-in containers, dict, list, set, and tuple.
## 基本介绍
我们都知道，Python 拥有一些内置的数据类型，比如 str, int, list, tuple, dict 等， collections 模块在这些内置数据类型的基础上，提供了几个额外的数据类型：
- namedtuple(): 生成可以使用名字来访问元素内容的 tuple 子类
- deque: 双端队列，可以快速的从另外一侧追加和推出对象
- Counter: 计数器，主要用来计数
- OrderedDict: 有序字典
- defaultdict: 带有默认值的字典
## namedtuple

工厂函数，用于创建具有命名字段的元组子类，通常用来增强代码的可读性， 在访问一些 tuple 类型的数据时尤其好用。

- 语法

```python
namedtuple(typename, field_names, *, verbose=False, rename=False, module=None)
```

- Example

```python
# 导入namedtuple
>>> from collections import namedtuple
# 创建一个用户类，拥有name,age,sex属性
>>> User = namedtuple("User", ["name","age","sex"])
>>> user1 = User("as","22","男")
>>> user1
User(name='as', age='22', sex='男')
# 通过属性进行访问
>>> user1.name
'as'
>>> user1.age
'22'
>>> user1.sex
'男'
# 拆包
>>> name, age, sex = user1
>>> name, age, sex
('as', '22', '男')
# namedtuple转换为字典
>>> user1._asdict()
OrderedDict([('name', 'as'), ('age', '22'), ('sex', '男')])
# 初始化时也可以传入一个字典
>>> user_info_dict = {"name":"ansheng","age":"20","sex":"男"}
>>> User(**user_info_dict)
User(name='ansheng', age='20', sex='男')
# 传入一个可迭代的
>>> user_info_list = ["ansheng","20","女"]
>>> User(*user_info_list)
User(name='ansheng', age='20', sex='女')
>>> user_info_list = ["ansheng","20"]
>>> User(*user_info_list,"男")
User(name='ansheng', age='20', sex='男')
# _make方法
>>> User._make(["as", 20, "男"])
User(name='as', age=20, sex='男')
```

## deque
### deque

deque 其实是 double-ended queue 的缩写，翻译过来就是双端队列，它最大的好处就是实现了从队列 头部快速增加和取出对象: .popleft(), .appendleft() 。

你可能会说，原生的 list 也可以从头部添加和取出对象啊？像这样：
```python
l.insert(0, v)
l.pop(0) 
```
但是值得注意的是，list 的这两种用法的**时间复杂度*是 O(n) ，也就是说随着元素数量的增加耗时呈 线性上升。而使用 deque 对象则是 O(1) 的复杂度，所以当你的代码有这样的需求的时候， 一定要记得使用 deque。

作为一个双端队列，deque 还提供了一些其他的好用方法，比如 rotate 等。

- Example1
```python
# -*- coding: utf-8 -*-
"""
下面这个是一个有趣的例子，主要使用了deque的rotate方法来实现了一个无限循环
的加载动画
"""
import sys
import time
from collections import deque

fancy_loading = deque('>--------------------')

while True:
    print 'r%s' % ''.join(fancy_loading),
    fancy_loading.rotate(1)
    sys.stdout.flush()
    time.sleep(0.08)

# Result:

# 一个无尽循环的跑马灯
------------->-------
```
- Example2

```python
>>> from collections import deque
>>> users_deque = deque(["user1", "user2", "user3"], maxlen=10)
>>> users_deque.appendleft("user4")
>>> users_deque.append("user4")
>>> users_deque
deque(['user4', 'user1', 'user2', 'user3', 'user4'], maxlen=10)
```

## ChainMap

dict-like class for creating a single view of multiple mappings

- Example

```python
>>> from collections import ChainMap
>>> dict1 = {"a": "a", "b": "b"}
>>> dict2 = {"c": "c", "d": "d"}
>>> new_dict = ChainMap(dict1, dict2)
>>> new_dict.maps
[{'a': 'a', 'b': 'b'}, {'c': 'c', 'd': 'd'}]
>>> for key, value in new_dict.items():
...     print(key, value)
...
a a
b b
d d
c c
```


## Counter

统计元素出现的次数

- Example

```python
>>> from collections import Counter
# 拥挤英文字母出现的次数
>>> letters = ["A", "A", "B", "C", "A", "H", "D", "B"]
>>> letters_counter = Counter(letters)
>>> letters_counter.update(["A", "A"])
>>> letters_counter
Counter({'A': 5, 'B': 2, 'C': 1, 'H': 1, 'D': 1})
# 出现最多的前2个元素
>>> letters_counter.most_common(2)
[('A', 5), ('B', 2)]
# 统计字符串
>>> Counter("asdasdsczasdasdasdasd")
Counter({'s': 7, 'a': 6, 'd': 6, 'c': 1, 'z': 1})
```


## OrderedDict

dict subclass that remembers the order entries were added
在 Python 中，dict 这个数据结构由于 hash 的特性，是无序的，这在有的时候会给我们带来一些麻烦， 幸运的是，collections 模块为我们提供了 OrderedDict，当你要获得一个有序的字典对象时，用它就对了。
{% note warning %}
在 python3.6 中字典已经有序了，参见：[[Python-Dev] Python 3.6 dict becomes compact and gets a private version; and keywords become ordered](https://mail.python.org/pipermail/python-dev/2016-September/146327.html)
{% endnote %}

- Example

```python
>>> from collections import OrderedDict
>>> user_dict = OrderedDict()
>>> user_dict["name"] = "As"
>>> user_dict["age"] = 18
>>> user_dict["sex"] = "男"
>>> user_dict
OrderedDict([('name', 'As'), ('age', 18), ('sex', '男')])
# 把name移动到最后
>>> user_dict.move_to_end("name")
>>> user_dict
OrderedDict([('age', 18), ('sex', '男'), ('name', 'As')])
# 移除最后一个元素
>>> user_dict.popitem()
('name', 'As')
>>> user_dict
OrderedDict([('age', 18), ('sex', '男')])
```

## defaultdict

当字典中 Key 不存在时，设置默认值

- 语法

```python
defaultdict(FUNC_NAME)
```

接受一个函数名称

- Code

统计列表中字母出现的次数

```python
from collections import defaultdict

letters = ["A", "A", "B", "C", "A", "H", "D", "B"]

data_dict = defaultdict(int)

for letter in letters:
    data_dict[letter] += 1
print(data_dict)
```

- Output

```plain
defaultdict(<class 'int'>, {'A': 3, 'B': 2, 'C': 1, 'H': 1, 'D': 1})
```

## UserDict

wrapper around dictionary objects for easier dict subclassing


## UserList

wrapper around list objects for easier list subclassing


## UserString

wrapper around string objects for easier string subclassing
