---
title: CentOS7 部署 nautilus 版 CEPH（使用 ceph-deploy）
tags:
  - CEPH
  - 环境搭建
  - CentOS7
cover: 'https://i1.wp.com/ceph.io/wp-content/uploads/2019/03/nautilus.svg_.png'
subtitle: 人生苦短，不要把时间浪费在重复性工作上。本文主要以 ceph-deploy 为例实践了 ceph 集群的部署流程。
categories:
  - "\U0001F4BB工作"
  - 存储
  - CEPH
  - 2-部署
date: 2020-03-15 12:37:35
---
## 引言
项目开发中准备基于 ceph-mgr 中的 dashboard 做二次开发，本文主要记录搭建 ceph 环境的过程。
### 环境说明
节点配置中参考官网搭建示例，基于实体机内存状况限制，使用三节点最小节点配置，具体如下：
![ceph 节点组成说明](/images/ceph-1.png)
## 前期准备
1. 配置网络为静态 ip
简略
2. 添加 CEPH 的 yum 源
```plain
vim /etc/yum.repos.d/ceph.repo
```
```plain
[Ceph]
name=Ceph packages for $basearch
baseurl=http://mirrors.aliyun.com/ceph/rpm-nautilus/el7/$basearch
enabled=1
gpgcheck=0
type=rpm-md

[Ceph-noarch]
name=Ceph noarch packages
baseurl=http://mirrors.aliyun.com/ceph/rpm-nautilus/el7/noarch
enabled=1
gpgcheck=0
type=rpm-md

[ceph-source]
name=Ceph source packages
baseurl=http://mirrors.aliyun.com/ceph/rpm-nautilus/el7/SRPMS
enabled=1
gpgcheck=0
type=rpm-md
```
3. 更新 epel.repo
国内使用阿里源加快下载
 ```plain
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
# 或者直接写入
[epel]
name=Extra Packages for Enterprise Linux 7 - $basearch
baseurl=http://mirrors.aliyun.com/epel/7/$basearch
failovermethod=priority
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7

[epel-debuginfo]
name=Extra Packages for Enterprise Linux 7 - $basearch - Debug
baseurl=http://mirrors.aliyun.com/epel/7/$basearch/debug
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=0

[epel-source]
name=Extra Packages for Enterprise Linux 7 - $basearch - Source
baseurl=http://mirrors.aliyun.com/epel/7/SRPMS
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=0
```plain
4. 更新源并安装 ceph-deploy
```plain
sudo yum update
sudo yum install ceph-deploy
```
5. 安装 NTP
```plain
sudo yum install ntp ntpdate ntp-doc
```
6. 安装 SSH 服务
```plain
sudo yum install openssh-server
```
7. 修改 host
```shell
vi /etc/hosts
```
```plain
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
172.18.1.101 admin-node
172.18.1.102 node1
172.18.1.103 node2
```

### 创建 CEPH DEPLOY 用户（以 cephadm 为例）
1. 创建用户
```plain
# 本例中使用 cephadm
sudo useradd -d /home/cephadm -m cephadm
sudo passwd cephadm
```
2. sudo 赋权
```plain
echo "cephadm ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/cephadm
sudo chmod 0440 /etc/sudoers.d/cephadm
```
3. tty
使用 sudo visudo 定位`Defaults requiretty`配置修改为`Defaults:ceph !requiretty`或者直接注释掉；
---
