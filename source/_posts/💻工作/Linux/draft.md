---
title: TODO，其他暂时无法归类
toc: true
tags:
  - 存储
categories:
  - "\U0001F4BB工作"
  - Linux
date: 2020-05-23 12:27:56
---
## 不重启扫描磁盘
```bash
echo '- - -' > /sys/class/scsi_host/host0/scan  # 有几个host就扫面几个，除非找到已加磁盘
```

## 指令查询
[jaywcjlove/linux-command: Linux命令大全搜索工具，内容包含Linux命令手册、详解、学习、搜集。https://git.io/linux](https://github.com/jaywcjlove/linux-command)