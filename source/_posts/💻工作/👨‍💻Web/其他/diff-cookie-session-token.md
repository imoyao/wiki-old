---
title: 如何理解 cookie、session 与 token
tags:
  - Web
  - cookie
  - session
categories:
  - "\U0001F4BB 工作"
  - "\U0001F468‍\U0001F4BBWeb"
  - 其他
date: 2019-06-18 14:06:27
---
## Cookie的机制

Cookie是浏览器（User Agent）访问一些网站后，这些网站存放在客户端的一组数据，用于使网站等跟踪用户，实现用户自定义功能。

Cookie的Domain和Path属性标识了这个Cookie是哪一个网站发送给浏览器的；Cookie的Expires属性标识了Cookie的有 效时间，当Cookie的有效时间过了之后，这些数据就被自动删除了。

如果不设置过期时间，则表示这个Cookie生命周期为浏览器会话期间，只要关闭浏览器窗口，Cookie就消失了。这种生命期为浏览会话期的 Cookie被称为会话Cookie。会话Cookie一般不保存在硬盘上而是保存在内存里。如果设置了过期时间，浏览器就会把Cookie保存到硬盘 上，关闭后再次打开浏览器，这些Cookie依然有效直到超过设定的过期时间。存储在硬盘上的Cookie可以在不同的浏览器进程间共享，比如两个IE窗 口。而对于保存在内存的Cookie，不同的浏览器有不同的处理方式。

 

## Session的机制

Session是存放在服务器端的类似于HashTable结构（每一种Web开发技术的实现可能不一样，下文直接称之为HashTable）来存放用户数据，当浏览器第一次发送请求时，服务器自动生成了一个HashTable和一个Session ID用来唯一标识这个HashTable，并将其通过响应发送到浏览器。当浏览器第二次发送请求，会将前一次服务器响应中的Session ID放在请求中一并发送到服务器上，服务器从请求中提取出Session ID，并和保存的所有Session ID进行对比，找到这个用户对应的HashTable。

一般情况下，服务器会在一定时间内（默认20分钟）保存这个HashTable，过了时间限制，就会销毁这个HashTable。在销毁之前，程序员可以 将用户的一些数据以Key和Value的形式暂时存放在这个HashTable中。当然，也有使用数据库将这个HashTable序列化后保存起来的，这 样的好处是没了时间的限制，坏处是随着时间的增加，这个数据库会急速膨胀，特别是访问量增加的时候。一般还是采取前一种方式，以减轻服务器压力。

## 参考阅读
[彻底弄懂 session，cookie，token](https://segmentfault.com/a/1190000017831088)
[token 与 sessionId 的区别——学习笔记](https://segmentfault.com/a/1190000015881055)
[彻底理解 cookie，session，token](https://www.liangzl.com/get-article-detail-16019.html)
[HTTP 是一个无状态的协议。这句话里的无状态是什么意思？](https://www.zhihu.com/question/23202402)
