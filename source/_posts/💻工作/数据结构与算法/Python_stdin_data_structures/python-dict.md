---
title: 字典
toc: true
tags:
  - 算法
categories:
  - "\U0001F4BB 工作"
  - 数据结构与算法
  - Python_stdin_data_structures
date: 2020-05-27 18:21:46
---
字典类型是 Python 中最常用的数据类型之一，它是一个键值对的集合。`dict`与`list`的不同之处在于你需要通过一个键（`key`）来访问元素，而不是通过`index`。
不过现在我们要说的重点是，`dict`条目的访问和赋值都是`O(1)`的时间复杂度。`dict`的另一个重要的操作是所谓的`in`。检查一个键是否存在于`dict`中也只需 `O(1)`的时间。

```python
\>>> d = {'a': 1, 'b': 2}
>>> d\['c'\] = 3
>>> d
{'a': 1, 'b': 2, 'c': 3}
```
在[字符串的实现原理](http://foofish.net/blog/91/python-str-implements)文章中，曾经出现过字典对象用于 intern 操作，那么字典的内部结构是怎样的呢？PyDictObject 对象就是 dict 的内部实现。

## 哈希表 (hash tables)

哈希表（也叫散列表），根据关键值对(Key-value)而直接进行访问的数据结构。它通过把 key 和 value 映射到表中一个位置来访问记录，这种查询速度非常快，更新也快。而这个映射函数叫做哈希函数，存放值的数组叫做哈希表。 哈希函数的实现方式决定了哈希表的搜索效率。具体操作过程是：

1.  数据添加：把 key 通过哈希函数转换成一个整型数字，然后就将该数字对数组长度进行取余，取余结果就当作数组的下标，将 value 存储在以该数字为下标的数组空间里。
2.  数据查询：再次使用哈希函数将 key 转换为对应的数组下标，并定位到数组的位置获取 value。

但是，对 key 进行 hash 的时候，不同的 key 可能 hash 出来的结果是一样的，尤其是数据量增多的时候，这个问题叫做哈希冲突。如果解决这种冲突情况呢？通常的做法有两种，一种是链接法，另一种是开放寻址法，Python 选择后者。

## 开放寻址法（open addressing）

开放寻址法中，所有的元素都存放在散列表里，当产生哈希冲突时，通过一个探测函数计算出下一个候选位置，如果下一个获选位置还是有冲突，那么不断通过探测函数往下找，直到找个一个空槽来存放待插入元素。

## PyDictEntry

字典中的一个 key-value 键值对元素称为 entry（也叫做 slots），对应到 Python 内部是 PyDictEntry，PyDictObject 就是 PyDictEntry 的集合。PyDictEntry 的定义是：
```c
typedef struct {
    /\* Cached hash code of me\_key.  Note that hash codes are C longs.
     \* We have to use Py\_ssize\_t instead because dict\_popitem() abuses
     \* me\_hash to hold a search finger.
     \*/
    Py\_ssize\_t me\_hash;
    PyObject \*me\_key;
    PyObject \*me\_value;
} PyDictEntry;
```
me\_hash 用于缓存 me\_key 的哈希值，防止每次查询时都要计算哈希值，entry 有三种状态。

1.  Unused： me\_key == me\_value == NULL

    Unused 是 entry 的初始状态，key 和 value 都为 NULL。插入元素时，Unused 状态转换成 Active 状态。这是 me\_key 为 NULL 的唯一情况。 2. Active： me\_key != NULL and me\_key != dummy 且 me\_value != NULL

    插入元素后，entry 就成了 Active 状态，这是 me\_value 唯一不为 NULL 的情况，删除元素时 Active 状态刻转换成 Dummy 状态。 3. Dummy： me\_key == dummy 且 me\_value == NULL

    此处的 dummy 对象实际上一个 PyStringObject 对象，仅作为指示标志。Dummy 状态的元素可以在插入元素的时候将它变成 Active 状态，但它不可能再变成 Unused 状态。


为什么 entry 有 Dummy 状态呢？这是因为采用开放寻址法中，遇到哈希冲突时会找到下一个合适的位置，例如某元素经过哈希计算应该插入到 A 处，但是此时 A 处有元素的，通过探测函数计算得到下一个位置 B，仍然有元素，直到找到位置 C 为止，此时 ABC 构成了探测链，查找元素时如果 hash 值相同，那么也是顺着这条探测链不断往后找，当删除探测链中的某个元素时，比如 B，如果直接把 B 从哈希表中移除，即变成 Unused 状态，那么 C 就不可能再找到了，因为 AC 之间出现了断裂的现象，正是如此才出现了第三种状态---Dummy，Dummy 是一种类似的伪删除方式，保证探测链的连续性。
![python_entry_status](http://img.foofish.net/python_entry_status.jpg)

## PyDictObject

PyDictObject 就是 PyDictEntry 对象的集合，PyDictObject 的结构是：
```c
typedef struct \_dictobject PyDictObject;
struct \_dictobject {
    PyObject\_HEAD
    Py\_ssize\_t ma\_fill;  /\* # Active + # Dummy \*/
    Py\_ssize\_t ma\_used;  /\* # Active \*/

    /\* The table contains ma\_mask + 1 slots, and that's a power of 2.
     \* We store the mask instead of the size because the mask is more
     \* frequently needed.
     \*/
    Py\_ssize\_t ma\_mask;

    /\* ma\_table points to ma\_smalltable for small tables, else to
     \* additional malloc'ed memory.  ma\_table is never NULL!  This rule
     \* saves repeated runtime null-tests in the workhorse getitem and
     \* setitem calls.
     \*/
    PyDictEntry \*ma\_table;
    PyDictEntry \*(\*ma\_lookup)(PyDictObject \*mp, PyObject \*key, long hash);
    PyDictEntry ma\_smalltable\[PyDict\_MINSIZE\];
};
```
*   ma\_fill ：所有处于 Active 以及 Dummy 的元素个数
*   ma\_used ：所有处于 Active 状态的元素个数
*   ma\_mask ：所有 entry 的元素个数（Active+Dummy+Unused）
*   ma\_smalltable：创建字典对象时，一定会创建一个大小为 PyDict\_MINSIZE==8 的 PyDictEntry 数组。
*   ma\_table：当 entry 数量小于 PyDict\_MINSIZE，ma\_table 指向 ma\_smalltable 的首地址，当 entry 数量大于 8 时，Python 把它当做一个大字典来处理，此刻会申请额外的内存空间，同时将 ma\_table 指向这块空间。
*   ma\_lookup：字典元素的搜索策略

PyDictObject 使用 PyObject\_HEAD 而不是 PyObject\_Var\_HEAD，虽然字典也是变长对象，但此处并不是通过 ob\_size 来存储字典中元素的长度，而是通过 ma\_used 字段。

## PyDictObject 的创建过程
```c
PyObject \*
PyDict\_New(void)
{
    register PyDictObject \*mp;
    if (dummy == NULL) { /\* Auto-initialize dummy \*/
        dummy = PyString\_FromString("<dummy key>");
        if (dummy == NULL)
            return NULL;
    }
    if (numfree) {
        mp = free\_list\[--numfree\];
        assert (mp != NULL);
        assert (Py\_TYPE(mp) == &PyDict\_Type);
        \_Py\_NewReference((PyObject \*)mp);
        if (mp->ma\_fill) {
            EMPTY\_TO\_MINSIZE(mp);
        } else {
            /\* At least set ma\_table and ma\_mask; these are wrong
               if an empty but presized dict is added to freelist \*/
            INIT\_NONZERO\_DICT\_SLOTS(mp);
        }
        assert (mp->ma\_used == 0);
        assert (mp->ma\_table == mp->ma\_smalltable);
        assert (mp->ma\_mask == PyDict\_MINSIZE - 1);
    } else {
        mp = PyObject\_GC\_New(PyDictObject, &PyDict\_Type);
        if (mp == NULL)
            return NULL;
        EMPTY\_TO\_MINSIZE(mp);
    }
    mp->ma\_lookup = lookdict\_string;
    return (PyObject \*)mp;
}
```
1.  初始化 dummy 对象
2.  如果缓冲池还有可用的对象，则从缓冲池中读取，否则，执行步骤 3
3.  分配内存空间，创建 PyDictObject 对象，初始化对象
4.  指定添加字典元素时的探测函数，元素的搜索策略

## 字典搜索策略
```c
static PyDictEntry \*
lookdict(PyDictObject \*mp, PyObject \*key, register long hash)
{
    register size\_t i;
    register size\_t perturb;
    register PyDictEntry \*freeslot;
    register size\_t mask = (size\_t)mp-\>ma\_mask;
    PyDictEntry \*ep0 = mp-\>ma\_table;
    register PyDictEntry \*ep;
    register int cmp;
    PyObject \*startkey;

    i = (size\_t)hash & mask;
    ep = &ep0\[i\];
    if (ep-\>me\_key == NULL || ep-\>me\_key == key)
        return ep;

    if (ep-\>me\_key == dummy)
        freeslot = ep;
    else {
        if (ep-\>me\_hash == hash) {
            startkey = ep-\>me\_key;
            Py\_INCREF(startkey);
            cmp = PyObject\_RichCompareBool(startkey, key, Py\_EQ);
            Py\_DECREF(startkey);
            if (cmp < 0)
                return NULL;
            if (ep0 == mp-\>ma\_table && ep-\>me\_key == startkey) {
                if (cmp > 0)
                    return ep;
            }
            else {
                /\* The compare did major nasty stuff to the
                 \* dict:  start over.
                 \* XXX A clever adversary could prevent this
                 \* XXX from terminating.
                 \*/
                return lookdict(mp, key, hash);
            }
        }
        freeslot = NULL;
    }

    /\* In the loop, me\_key == dummy is by far (factor of 100s) the
       least likely outcome, so test for that last. \*/
    for (perturb = hash; ; perturb >>= PERTURB\_SHIFT) {
        i = (i << 2) + i + perturb + 1;
 ep = &ep0\[i & mask\];
 if (ep->me\_key == NULL)
            return freeslot == NULL ? ep : freeslot;
        if (ep-\>me\_key == key)
            return ep;
        if (ep-\>me\_hash == hash && ep-\>me\_key != dummy) {
            startkey = ep-\>me\_key;
            Py\_INCREF(startkey);
            cmp = PyObject\_RichCompareBool(startkey, key, Py\_EQ);
            Py\_DECREF(startkey);
            if (cmp < 0)
                return NULL;
            if (ep0 == mp-\>ma\_table && ep-\>me\_key == startkey) {
                if (cmp > 0)
                    return ep;
            }
            else {
                /\* The compare did major nasty stuff to the
                 \* dict:  start over.
                 \* XXX A clever adversary could prevent this
                 \* XXX from terminating.
                 \*/
                return lookdict(mp, key, hash);
            }
        }
        else if (ep-\>me\_key == dummy && freeslot == NULL)
            freeslot = ep;
    }
    assert(0);          /\* NOT REACHED \*/
    return 0;
}
```
字典在添加元素和查询元素时，都需要用到字典的搜索策略，搜索时，如果不存在该 key，那么返回 Unused 状态的 entry，如果存在该 key，但是 key 是一个 Dummy 对象，那么返回 Dummy 状态的 entry，其他情况就表示存在 Active 状态的 entry，那么对于字典的插入操作，针对不同的情况进行操作也不一样。对于 Active 的 entry，直接替换 me\_value 值即可；对于 Unused 或 Dummy 的 entry，需要同时设置 me\_key，me\_hash 和 me\_value

## PyDictObject 对象缓冲池

PyDictObject 对象缓冲池和 PyListObject 对象缓冲池的原理是类似的，都是在对象被销毁的时候把该对象添加到缓冲池中去，而且值保留 PyDictObject 对象本身，如果 ma\_table 维护的时从系统堆中申请的空间，那么 Python 会释放这块内存，如果 ma\_table 维护的是 ma\_smalltable，那么只需把 smalltable 中的元素的引用计数减少即可。
```c
static void
dict\_dealloc(register PyDictObject \*mp)
{
    register PyDictEntry \*ep;
    Py\_ssize\_t fill = mp->ma\_fill;
    PyObject\_GC\_UnTrack(mp);
    Py\_TRASHCAN\_SAFE\_BEGIN(mp)
    for (ep = mp->ma\_table; fill > 0; ep++) {
        if (ep->me\_key) {
            --fill;
            Py\_DECREF(ep->me\_key);
            Py\_XDECREF(ep->me\_value);
        }
    }
    if (mp->ma\_table != mp->ma\_smalltable)
        PyMem\_DEL(mp->ma\_table);
    if (numfree < PyDict\_MAXFREELIST && Py\_TYPE(mp) == &PyDict\_Type)
        free\_list\[numfree++\] = mp;
    else
        Py\_TYPE(mp)->tp\_free((PyObject \*)mp);
    Py\_TRASHCAN\_SAFE\_END(mp)
}
```

## `dict`内置操作的时间复杂度

| 操作           | 操作说明 | 时间复杂度 |
| ---------------- | --------- | ---------- |
| copy             | 复制    | O(n)       |
| get(value)       | 获取    | O(1)       |
| set(value)       | 修改    | O(1)       |
| delete(value)    | 删除    | O(1)       |
| item `in` dict_obj | `in`关键字 | O(1)       |
| iterration       | 迭代 | O(n)       |


## 参考来源
[Python 字典对象实现原理 - FooFish-Python 之禅](https://foofish.net/python_dict_implements.html)
