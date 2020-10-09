---
title: Ceph 错误收集记录
tags:
  - ceph
  - processing
categories:
  - "\U0001F4BB工作"
  - 存储
  - CEPH
  - troubleshooting
subtitle: "想想说人生无悔，都是赌气的话。人生若无悔，那该多无趣啊。<br>所以说程序运行遇到报错是很正常的一件事。\U0001F61C"
cover: /images/sarah-kilian-52jRtc2S_VE-unsplash.jpg
date: 2020-03-16 20:41:54
---
## [errno 2] error connecting to the cluster
### 解释
安装 ceph 集群之后执行`ceph -s`报错如上，这个是因为认证文件没有分发到个节点导致的无法认证。
### 解决方案
```plain
ceph-deploy admin admin-node node1 [noden] # 后面跟你集群中的所有节点名
```
## daemons have recently crashed
### 解释
一个或多个 Ceph 守护进程最近崩溃了，管理员还没有存档（确认）这个崩溃。这可能表示软件错误、硬件问题(例如，故障磁盘)或其他问题。
### 解决方案
1. 查看 crash 信息
```plain
ceph crash ls-new
```
2. 查看归档信息
```plain
ceph crash info <crash-id>
```
3. 归档 crash 信息
```plain
ceph crash archive <crash-id>
```
你也可以使用`ceph crash archive-all`命令归档 所有信息
更多参考：
- [Crash Module — Ceph Documentation](https://docs.ceph.com/docs/master/mgr/crash/)
- [ceph 报 daemons have recently crashed_网络_lyf0327 的博客-CSDN 博客](https://blog.csdn.net/lyf0327/article/details/103315698/)
## xx is not defined in `mon initial members`
执行`ceph-deploy mon add xx`添加 mon 节点时报错如下：
```plain
[node3][WARNIN] node3 is not defined in `mon initial members`
[node3][WARNIN] monitor node3 does not exist in monmap
```
### 解释
添加的 mon 节点不是初始化成员，集群无法识别
### 解决方案
修改配置文件添加`public network = 172.18.1.0/24`（网段需要根据实际情况修改）并将配置文件同步到新加节点
```plain
ceph-deploy --overwrite-conf config push xxx
```
### 参考
- [Ceph: mon is down and/or can’t rejoin the quorum – swami reddy](https://swamireddy.wordpress.com/2017/09/20/ceph-mon-is-down-andor-cant-rejoin-the-quorum/)
- [ceph 在扩展 mon 节点时，要注意的问题 - aguncn - 博客园](https://www.cnblogs.com/aguncn/p/7352393.html)
- [Ceph 添加监视器 Monitor 失败_运维_weixin_33924220 的博客-CSDN 博客](https://blog.csdn.net/weixin_33924220/article/details/92602783)