---
title: TODO
toc: true
tags: others
categories:
  - "\U0001F4BB 工作"
  - "\U0001F40DPython"
  - Flask
date: 2020-05-26 12:27:56
---
## 文档
[欢迎来到 Flask 的世界 — Flask 中文文档（ 1.1.1 ）](https://dormousehole.readthedocs.io/en/latest/)

## 应用工厂
- [配置处理](http://www.pythondoc.com/flask/config.html)
- [大型应用程序结构](https://segmentfault.com/a/1190000002411388)

## 代码片段

### 生成随机字符串
- python2
 ```python
import random,string
# py2
In [14]: ''.join(random.choice(string.ascii_letters + string.digits) for _ in range(15))
Out[14]: 'B4X0a1MXZlq8PKb'
 ```plain
- python3
 ```python
import random
# py3.6+
''.join(random.choices(string.ascii_letters + string.digits, k=15))
# or
import secrets
secrets.token_urlsafe(nbytes=15)
 ```