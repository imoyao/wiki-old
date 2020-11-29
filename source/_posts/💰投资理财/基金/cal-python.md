---
title: 使用 Python 计算相关
tags:
  - 投资
  - 理财
categories:
  - 基金
date: 2020-11-29 12:27:56
---

## IRR
````python
import numpy


profile = numpy.irr([10000, 10000, 10000, 10000, 10000, 10000, 10000, 10000, 10000, 10000, 10000, 10000, -140000])
print (profile)
print(pow(profile+1, 12) - 1)
````
参见：
[python IRR公式使用——定投基金收益率计算_huiguixian的专栏-CSDN博客](https://blog.csdn.net/huiguixian/article/details/90714331)
[How to calculate IRR in Python :: Coding Finance](https://www.codingfinance.com/post/2018-03-20-irr-py/)