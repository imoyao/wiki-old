---
title: "FIFO | 待学清单\U0001F4DD"
tags:
  - TODO
  - 待学清单
cover: >-
  https://cdn.jsdelivr.net/gh/masantu/statics/image/jessica-lewis-fJXv46LT7Xk-unsplash.jpg
subtitle: "人人都有松鼠癖，人人都是马来人。\U0001F611"
top: 10
categories:
  - "\U0001F4BB工作"
  - "\U0001F40DPython"
date: 2019-11-27 23:34:31
---
- [x] RabbitMQ 官方文档
- [ ] Docker
- [x] 线程池、进程池
- [ ] 高并发框架
    - [Python 也能高并发](https://blog.51cto.com/youerning/2161196)
    - [Python 高并发编程](https://www.cnblogs.com/wsjhk/p/8502892.html)
    - [深入理解 Python 异步编程（上） - 简书](https://www.jianshu.com/p/fe146f9781d2)
- [ ] 数据库索引
    - [数据库索引到底是什么，是怎样工作的？](https://blog.csdn.net/weiliangliang111/article/details/51333169)
    - [How do database indexes work? And, how do indexes help? Provide a tutorial on database indexes.](https://www.programmerinterview.com/database-sql/what-is-an-index/)
- [x] redis 布隆过滤器
    - [redis 专题 06 布隆过滤器](https://researchlab.github.io/2018/10/03/redis-06-bloom-filter/)
    - [Redis 高级主题之布隆过滤器(BloomFilter)](https://juejin.im/post/5cfd060ee51d4556f76e8067)
    - [基于 Redis 的布隆过滤器的实现](https://blog.csdn.net/qq_30242609/article/details/71024458)
    - [布隆过滤器(Bloom Filter)的原理和实现](https://www.cnblogs.com/cpselvis/p/6265825.html)
- [ ] 设计模式[Design Patterns Book](http://wiki.c2.com/?DesignPatternsBook)
- [ ] 偏函数
 [python - How does functools partial do what it does? - Stack Overflow](https://stackoverflow.com/questions/15331726/how-does-functools-partial-do-what-it-does)
 [functional programming - Python: Why is functools.partial necessary? - Stack Overflow](https://stackoverflow.com/questions/3252228/python-why-is-functools-partial-necessary)
 - [ ] assert 
 [notes/when-to-use-assert.md at master · emre/notes](https://github.com/emre/notes/blob/master/python/when-to-use-assert.md)
 
## 代码风格
[Python 重构代码的一些模式 | Slient Plant](https://mpwang.github.io/2017/08/26/python-refactor-patterns/)

## debug
```python
import sys

def get_cur_info():
    print(sys._getframe().f_code.co_filename)  # 当前文件名
    print(sys._getframe(0).f_code.co_name)  # 当前函数名
    print(sys._getframe(1).f_code.co_name)　# 调用该函数的函数的名字，如果没有被调用，则返回module
    print(sys._getframe().f_lineno) # 当前行号
```
[Python 程序如何高效地调试？ - 知乎](https://www.zhihu.com/question/21572891/answer/123220574)
[打印日志 (log) 是比单步跟踪 (debugger) 更好的 Python 排错手段吗？ - 知乎](https://www.zhihu.com/question/20626825)

## 计算机书籍及知识体系
（不会真有人看完了吧？）
[NGTE Books](https://ng-tech.icu/books/)
## WSGI
[python wsgi 简介 | Cizixs Write Here](https://cizixs.com/2014/11/08/understand-wsgi/)
