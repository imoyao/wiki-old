---
title: drbdadm 中文手册
toc: true
tags:
  - DRBD
  - drbdadm
  - 存储
categories:
  - "\U0001F4BB工作"
  - 存储
  - DRBD
date: 2018-09-16 12:27:56
---
## 描述

drbdadm - DRBD 高级管理工具

## 指令

```plain
drbdadm [-d] [-c {_file_}] [-t {_file_}] [-s {_cmd_}] [-m {_cmd_}] [-S] [-h {_host_}] [-- {_backend-options_}] {_command_} [{all} | {_resource__[/volume>]_...}]
```
## 说明

`drbdadm`是 DRBD 程序中的高级管理工具。 `drbdadm`是`drbdsetup`和`drbdmeta`的较高级接口。 这类似于`ifconfig`与`ifup/ifdown`命令的关系。 `drbdadm`读取配置文件，然后调用`drbdsetup`或`drbdmeta`以运行命令。

`drbdadm`可以对所有资源和资源中的卷执行操作。 子命令包括： `attach`, `detach`, `primary`, `secondary`, `invalidate`, `invalidate-remote`, `outdate`, `resize`, `verify`, `pause-sync`, `resume-sync`, `role`, `csytate`, `dstate`, `create-md`, `show-gi`, `get-gi`, `dump-md`, `wipe-md`等，它们适用于所有资源和卷。

仅资源级命令如下所示： `connect`, `disconnect`, `up`, `down`, `wait-connect` 和 `dump`.

## 选项

`-d`， `--dry-run`

将执行的`drbdsetup`命令写入标准输出，但不实际运行该命令。

`-c`， `--config-file` *file*

指定 drbdadm 使用的配置文件。 如果省略此参数，则引用 /etc/drbd-84.conf、/etc/drbd-83.conf、/etc/drbd-08.conf、/etc/drbd.conf。

`-t`， `--config-to-test` *file*

指定要检查的配置文件。 仅在与转储或 sh-nop 命令一起使用时有效。

`-s`， `--drbdsetup` *file*

指定`drbdsetup`程序的完整路径。 如果省略，则引用自己的命令位置和$PATH。

`-m`， `--drbdmeta` *file*

指定`drbdmeta`程序的完整路径。 如果省略，则引用自己的命令位置和$PATH。

`-S`， `--stacked`

指定指示对堆叠的父资源的操作。

`-P`， `--peer`

指定要连接到的相反节点。 仅当为资源定义指定了三个或更多主机时，才需要它。

`--backend-options`

双连字符后面的所有选项都被视为*后端选项*。 它们传递给后端命令。 换句话说，它传递给`drbdsetup`、`drbdmeta`、`drbd-proxy-ctl`等。

## 指令

- attach

连接与 DRBD 资源对应的低级本地块设备。

- detach

将从属存储设备与 DRBD 资源设备分离。

- connect

为资源设备启用网络设置。 如果已配置相应的目标，则两个 DRBD 设备将相互连接。 如果在资源定义期间指定了三个或更多主机，则还必须指定`--peer`来指定要连接到的主机。

- disconnect

禁用资源的网络设置。 当然，设备将处于独立状态。

- syncer

加载设备重新同步的参数。

- up

同时执行连接和连接的快捷方式。

- down

同时执行连接和连接的快捷方式。

- primary

将资源设备提升为主状态。 在 DRBD 管理的设备中创建文件系统或装载文件系统之前，应始终运行此命令。

- secondary

将设备切换到辅助状态。 此命令是必需的，因为只有一个连接的 DRBD 设备对可以处于主状态（除非在配置文件中显式指定**了允许二次提示**）。

- invalidate

让 DRBD 确定_本地_从属设备不一致。 因此，DRBD 从另一个节点复制所有块，以便两个设备保持同步状态。 在这种情况下，复制链接必须具有概率，否则将处于断开连接的第二个状态。

- invalidate-remote

与无效命令类似，但_认为相反_节点的从属设备处于异步状态。 因此，本地节点中的数据将复制到其他节点。 在这种情况下，复制链接必须具有概率，否则将处于断开连接的"优先级"状态。

- resize

让 DRBD 重新评估与磁盘大小相关的状态，并在必要时调整设备的大小。 例如，如果两个节点上扩展从属设备的大小，则在其中一个节点上运行此命令后，DRBD 将接受新大小。 由于新的存储区域需要同步，因此仅当至少有一个主节点时，此命令才有效。

以下所有选项都是**反向选项**（由"-"分隔）：

**--size**选项用于在线减小 DRBD 设备可用的大小。 用户有责任确保文件系统不会因此操作而损坏。 示例：
```bash
# drbdadm -- --size=10G resize r0
```
**--assume-peer-has-space**选项允许您调整当前未连接到目标节点的设备的大小。 请注意，如果不以同样的方式更改相反节点的磁盘大小，则后续连接将失败。

**--assume-clean**选项允许您调整现有设备的大小，并避免同步新区域。 如果要向设备添加空存储，则非常有用。 示例：

```bash
# drbdadm -- --assume-clean resize r0
```
**--al-stripes** 和 **--al-stripe-size-kB** 选项在线更改活动日志的布局。 对于内部元数据，需要同时缩小或扩展子设备的用户可见的大小（使用**--size）。**

- check-resize

调用 drbdmeta 移动内部元数据。 如果在 DRBD 停止期间调整从属设备的大小，则必须将元数据移动到设备的末尾，以便下一个**附加**命令成功。

- create-md

初始化元数据区域。 如果这是您第一次使用 DRBD 资源，则需要在联机之前运行此命令。 如果出现问题，请参阅以下文档[：dbdmeta（8）](https://manpages.debian.org/testing/drbd-utils/drbdmeta.8.ja.html)

- get-gi

将数据生成标识符信息显示为简洁的文本信息。

- show-gi

将数据生成标识符信息与描述性文本一起显示为文本信息。

- dump-md

以纯文本形式转储元数据的全部内容。 转储还包括位图和活动日志。

- outdate

元数据具有 outdated 标志。

- adjust

根据配置文件的设置调整设备的设置状态。 在实际运行之前，应运行 dry-run 模式，以检查生成的输出。

- wait-connect

等待，直到连接到其他节点上的设备。

- role

以"机器/面向节点"的形式显示设备在机器和对台节点上的当前角色。 示例，主要/第二部分

- state

已弃用的"角色"别名。 参见前款规定。

- cstate

显示两个节点上设备的连接状态。

- dump

分析配置文件并将其输出到标准输出。 可用于修改配置文件的语法。

- outdate

使节点的数据状态无效。 通常由其他节点的 fence-peer 处理程序设置。

- verify

开始联机匹配。 比较两个节点的数据，并检查是否存在不一致。 进度显示在 /proc/drbd 中。 即使找到异步块，也**不会**自动重新同步。 要同步，请在检查完成后**断开连接**，然后**断开连接**。

另请参阅 drbd.conf 手册页的数据完整性说明。

-  pause-sync

设置本地元数据的暂停标志以暂停正在进行的重新同步。 若要恢复，必须清除本地节点和其他节点的暂停标志。 例如，如果正在重新配置从属设备的 RAID，则可以暂时停止 DRBD 重新同步。

- resume-sync

清除机器的暂停标志。

- new-current-uuid

生成新的当前 UUID 并旋转所有其他 UUID。

此命令可用于缩短初始同步时间。 有关详细信息**，请参阅 drbdsetup**手册页。

- dstate

显示从属设备的同步状态。 （local/peer）

- hidden-commands

显示本文档中未列出的所有命令。

## 版本

本文档已针对 DRBD 版本 8.4.0 进行了修订。

## 作者


philipp.Reisner <philipp.reisner@linbit.com> lars.ellenberg <lars.ellenberg@linbit.com>

## 报告 BUGS

报告 bugs 给 <drbd-user@lists.linbit.com>。

## COPYRIGHT

Copyright 2001-2011 LINBIT Information Technologies, Philipp Reisner, Lars Ellenberg. This is free software; see the source for copying conditions. There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

## 参见

[drbd.conf(5)](https://manpages.debian.org/testing/drbd-utils/drbd.conf.5.en.html), [drbd(8)](https://manpages.debian.org/testing/drbd-utils/drbd.8.en.html), [drbddisk(8)](https://manpages.debian.org/testing/drbd-utils/drbddisk.8.en.html), [drbdsetup(8)](https://manpages.debian.org/testing/drbd-utils/drbdsetup.8.en.html), [drbdmeta(8)](https://manpages.debian.org/testing/drbd-utils/drbdmeta.8.en.html) and the **DRBD project web site**[1]
