---
title: Redis 哨兵模式配置
tags:
  - Redis
  - NoSQL
  - 数据库
  - HOWTO
toc: true
categories:
  - "\U0001F4BB 工作"
  - 数据库
  - redis
date: 2019-04-11 10:03:08
---
## 配置 master
- 切换目录
    ```shell
    pwd
    /etc/redis
    ```

- 复制配置

    ```shell
    cp 6379.conf master.conf
    ```

- 编辑配置

    1. 配置网络
        ```shell
        # nu:69
        # bind 127.0.0.1
        bind 172.18.1.110   # ifconfig 查看
        port 6379
        timeout 30
        ```
    2. GENERAL 设置：
        ```shell
        daemonize yes    //默认值是no，把值修改为yes，以后台模式运行
        logfile /var/log/redis/redis-server.log  //日志文件的位置
        ```
    3. SNAPSHOTTING 设置
        ```shell
        dir /var/lib/redis/data     //SNAPSHOTTING文件的路径
        ```

## 配置 slave

操作同上
- 复制配置

    ```shell
    cp 6379.conf slave.conf
    ```
- 编辑配置
    - 编辑网络
        ```shell
        bind 172.18.1.111    # ifconfig 查看
        port 6379
        timeout 30
        ```
    - REPLICATION 设置
     ```shell
        # nu:289
        # replicaof <masterip> <masterport>
        replicaof 172.18.1.110 6379
        replica-serve-stale-data no  //此处按需设置，no为如果 slave 无法与master 同步，设置成slave不可读，方便监控脚本发现问题。
     ```
    **注意**：旧版本为`slaveof`,新版由于“政治正确”改为`replicaof`，人生总是充满无奈啊！

    - [issues:5335>>Changing Redis master-slave replication terms with something else](https://github.com/antirez/redis/issues/5335)

    - [On Redis master-slave terminology](http://antirez.com/news/122)
    
## 配置哨兵

- 复制配置文件
    ```shell
    cp /home/imoyao/temp/redis-5.0.4/sentinel.conf ./  # 源目录决定于初始解压到了哪里
    ```
- 编辑 port
    ```shell
    port 26379  //哨兵端口号保持不变，可以修改，但是我没有修改
    ```
- 编辑`bind`或者`protected-mode`
    建议此处详细阅读配置文件部分，理解之后哨兵模式配置应无虞。
    注意此处要和`server`的配置对应上，`bind`不一定要写成`0.0.0.0`，网上很多文章这点说的都是不准确的。设置成`0.0.0.0`感觉失去`bind`意义了。
    ```shell
    protected-mode no
    ```
- SNAPSHOTTING 设置
    ```shell 
    # +65
    dir /var/lib/redis/sentinel
    ```
- monitor 配置
    客观下线依据
    ```shell 
    # sentinel monitor <master-name> <ip> <redis-port> <quorum>
    # 配置监听的主服务器，这里sentinel monitor代表监控，mymaster代表服务器的名称，可以自定义，192.168.11.128代表监控的主服务器，6379代表端口，2代表只有两个或两个以上的哨兵认为主服务器不可用的时候，才会进行failover操作。
    sentinel monitor mymaster 192.168.11.128 6379 1
    ```

- down-after-milliseconds 配置
    主观下线依据

    ```shell
    # sentinel down-after-milliseconds <master-name> <milliseconds>
    sentinel down-after-milliseconds mymaster 5000         
    ```

- parallel-syncs 配置
    故障迁移时，有几个从端提供查询服务，使用小数字，避免同步时无法访问

    ```shell
    # sentinel parallel-syncs <master-name> <numreplicas>  # ns
    sentinel parallel-syncs mymaster 1
    ``` 
- failover-timeout 配置
    ```shell
     # sentinel failover-timeout <master-name> <milliseconds>
     sentinel failover-timeout mymaster 60000
    ```

## 启动 master
```shell
redis-server /etc/redis/master.conf
```
## 启动客户端验证
```shell
redis-cli 
```
此时会报错

```shell
Could not connect to Redis at 127.0.0.1:6379: Connection refused
```
- 指定 ip 和端口
    ```shell
    redis-cli -h 172.18.1.110 -p 6379
    
    # // out
    172.18.1.110:6379> info
    # Server
    redis_version:5.0.4
    redis_git_sha1:00000000
    redis_git_dirty:0
    redis_build_id:e0545b56586aeb37
    redis_mode:standalone
    os:Linux 2.6.32-431.1.ky3.1.x86_64 x86_64
    arch_bits:64
    multiplexing_api:epoll
    atomicvar_api:sync-builtin
    xxx     # TL;NW
    
    ```
- 查看具体的部分配置
    ```shell
    172.18.1.110:6379> info replication
    # //out
    
    # Replication
    role:master     # 本机为主端
    connected_slaves:0
    master_replid:042d60232c761b4acbe268a1da18ef9c830463df
    master_replid2:0000000000000000000000000000000000000000
    master_repl_offset:0
    second_repl_offset:-1
    repl_backlog_active:0
    repl_backlog_size:1048576
    repl_backlog_first_byte_offset:0
    repl_backlog_histlen:0
    ```
## 启动备机

```shell
redis-cli -h 172.18.1.111  -p 6379

# out
172.18.1.111:6379> info replication
# Replication
role:slave
master_host:172.18.1.110
master_port:6379
master_link_status:up
master_last_io_seconds_ago:1
master_sync_in_progress:0
slave_repl_offset:334327
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:6f9d6e03a1580e68a94e05b3de7a2cc3632e169e
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:334327
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:334327
```

## 启动哨兵守护进程

### 从机启动哨兵服务

```shell
root@local:/etc/redis# redis-sentinel ./sentinel.conf 

3268:X 24 Apr 2019 15:43:04.243 # Can't chdir to '/var/lib/redis/sentinel': No such file or directory       # 无目录
root@local:/etc/redis# mkdir /var/lib/redis/sentinel            # 创建目录
root@local:/etc/redis# redis-sentinel ./sentinel.conf           # 再次启动
3270:X 24 Apr 2019 15:43:19.684 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
3270:X 24 Apr 2019 15:43:19.684 # Redis version=5.0.4, bits=64, commit=00000000, modified=0, pid=3270, just started
3270:X 24 Apr 2019 15:43:19.684 # Configuration loaded
```
### 验证

```shell
root@local:/etc/redis# ps aux|grep redis
root      3225  1.8 50.4 1433480 1030592 ?     Ssl  15:37   0:06 redis-server 172.18.1.111:6379
root      3271  0.5  0.2  61320  4456 ?        Ssl  15:43   0:00 redis-sentinel *:26379 [sentinel]
```
### 主机启动哨兵

略

#### 验证

```shell
[root@master redis]# ps aux|grep redis
root     22479  0.1  0.2 158040  8112 ?        Ssl  15:19   0:04 redis-server 172.18.1.110:6379
root     27079  0.2  0.2 152412  7952 ?        Ssl  15:54   0:00 redis-sentinel *:26379 [sentinel]      
root     27405  0.0  0.0 103248   884 pts/0    S+   15:54   0:00 grep redis
root     27737  0.0  0.2  23996  7192 pts/2    S+   15:36   0:00 redis-cli -h 172.18.1.110 -p 6379
```

### 从机登录客户端
    
```shell
172.18.1.111:26379> info sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=172.18.1.110:6379,slaves=1,sentinels=2
    ```

### 主机登录客户端

```shell
172.18.1.110:26379> info sentinel

# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=172.18.1.110:6379,slaves=1,sentinels=2

```
## 模拟主机宕机（手动 kill）

```shell
# 方案1
ps aux |grep redis |grep -v grep
找到server对应的端口
kill -9 xxx(id号)

# 方案2
redis-cli shutdown
```

### 查看正常切换后的哨兵日志
    
```shell
tail -f /var/log/redis/redis-sentinel.log
```

```shell
26661:X 25 Apr 2019 09:22:18.923 # +new-epoch 1         # 仲裁开始
26661:X 25 Apr 2019 09:22:18.929 # +vote-for-leader 4a818323037cbd9aef1d2d6fd7a2257119308adb 1   # 投票
26661:X 25 Apr 2019 09:22:18.941 # +sdown master root 172.18.1.110 6379
26661:X 25 Apr 2019 09:22:18.941 # +odown master root 172.18.1.110 6379 #quorum 1/1
26661:X 25 Apr 2019 09:22:18.941 # Next failover delay: I will not start a failover before Thu Apr 25 09:24:19 2019
26661:X 25 Apr 2019 09:22:19.322 # +config-update-from sentinel 4a818323037cbd9aef1d2d6fd7a2257119308adb 172.18.1.110 26379 @ root 172.18.1.110 6379
26661:X 25 Apr 2019 09:22:19.322 # +switch-master root 172.18.1.110 6379 172.18.1.111 6379   # 切换
26661:X 25 Apr 2019 09:22:19.322 * +slave slave 172.18.1.110:6379 172.18.1.110 6379 @ root 172.18.1.111 6379
26661:X 25 Apr 2019 09:22:24.328 # +sdown slave 172.18.1.110:6379 172.18.1.110 6379 @ root 172.18.1.111 6379

```
### 登录(原来的)从客户端验证

```shell
172.18.1.111:6379> info replication
# Replication
role:master    # slave >>> master
connected_slaves:0
master_replid:a9abe72647d6924d84f06e12f0bc26c6dc7e057c
master_replid2:2a17e5cc5502cea4c063952fc3892348c56119be
master_repl_offset:60882
second_repl_offset:58430
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:60882
172.18.1.111:6379> 
```
### 登录(原来的)主客户端验证

```shell
172.18.1.110:6379> info replication
# Replication
role:slave
master_host:172.18.1.111
master_port:6379
master_link_status:up
master_last_io_seconds_ago:1
master_sync_in_progress:0
slave_repl_offset:115729
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:a9abe72647d6924d84f06e12f0bc26c6dc7e057c
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:115729
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:114343
repl_backlog_histlen:1387

```
### （原）主机日志

```shell
10541:M 25 Apr 2019 21:29:10.110 * Ready to accept connections
10541:S 25 Apr 2019 21:29:20.515 * Before turning into a replica, using my master parameters to synthesize a cached master: I may be able to synchronize with the new master with just a partial transfer.
10541:S 25 Apr 2019 21:29:20.515 * REPLICAOF 172.18.1.111:6379 enabled (user request from 'id=5 addr=172.18.1.111:50750 fd=9 name=sentinel-a049bf2a-cmd age=10 idle=0 flags=x db=0 sub=0 psub=0 multi=3 qbuf=152 qbuf-free=32616 obl=36 oll=0 omem=0 events=r cmd=exec')
10541:S 25 Apr 2019 21:29:20.522 # CONFIG REWRITE executed with success.
10541:S 25 Apr 2019 21:29:21.249 * Connecting to MASTER 172.18.1.111:6379
10541:S 25 Apr 2019 21:29:21.249 * MASTER <-> REPLICA sync started
10541:S 25 Apr 2019 21:29:21.249 * Non blocking connect for SYNC fired the event.
10541:S 25 Apr 2019 21:29:21.251 * Master replied to PING, replication can continue...
10541:S 25 Apr 2019 21:29:21.252 * Trying a partial resynchronization (request c7eaf39dc5c013646804d1ba35b6aeb91183dbfc:1).
10541:S 25 Apr 2019 21:29:21.331 * Full resync from master: a9abe72647d6924d84f06e12f0bc26c6dc7e057c:114342
10541:S 25 Apr 2019 21:29:21.331 * Discarding previously cached master state.
10541:S 25 Apr 2019 21:29:21.432 * MASTER <-> REPLICA sync: receiving 178 bytes from master
10541:S 25 Apr 2019 21:29:21.433 * MASTER <-> REPLICA sync: Flushing old data
10541:S 25 Apr 2019 21:29:21.433 * MASTER <-> REPLICA sync: Loading DB in memory
10541:S 25 Apr 2019 21:29:21.433 * MASTER <-> REPLICA sync: Finished with success

```
### （现）主机日志
重写配置，同步
```shell
26655:M 25 Apr 2019 09:22:19.155 # Setting secondary replication ID to 2a17e5cc5502cea4c063952fc3892348c56119be, valid up to offset: 58430. New replication ID is a9abe72647d6924d84f06e12f0bc26c6dc7e057c
26655:M 25 Apr 2019 09:22:19.155 * Discarding previously cached master state.
26655:M 25 Apr 2019 09:22:19.155 * MASTER MODE enabled (user request from 'id=9 addr=172.18.1.110:42186 fd=8 name=sentinel-4a818323-cmd age=122 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=140 qbuf-free=32628 obl=83 oll=0 omem=0 events=r cmd=slaveof')
26655:M 25 Apr 2019 09:22:19.158 # CONFIG REWRITE executed with success.
26655:M 25 Apr 2019 09:29:21.256 * Replica 172.18.1.110:6379 asks for synchronization
26655:M 25 Apr 2019 09:29:21.256 * Partial resynchronization not accepted: Replication ID mismatch (Replica asked for 'c7eaf39dc5c013646804d1ba35b6aeb91183dbfc', my replication IDs are 'a9abe72647d6924d84f06e12f0bc26c6dc7e057c' and '2a17e5cc5502cea4c063952fc3892348c56119be')
26655:M 25 Apr 2019 09:29:21.256 * Starting BGSAVE for SYNC with target: disk
26655:M 25 Apr 2019 09:29:21.261 * Background saving started by pid 26693
26693:C 25 Apr 2019 09:29:21.348 * DB saved on disk
26693:C 25 Apr 2019 09:29:21.349 * RDB: 4 MB of memory used by copy-on-write
26655:M 25 Apr 2019 09:29:21.435 * Background saving terminated with success
26655:M 25 Apr 2019 09:29:21.436 * Synchronization with replica 172.18.1.110:6379 succeeded
```
### 哨兵日志
```shell
26661:X 25 Apr 2019 09:29:10.599 # -sdown slave 172.18.1.110:6379 172.18.1.110 6379 @ root 172.18.1.111 6379
26661:X 25 Apr 2019 09:29:20.518 * +convert-to-slave slave 172.18.1.110:6379 172.18.1.110 6379 @ root 172.18.1.111 6379   # 数据合并
```

## 异常

- 一直在进行投票，但是故障迁移（`failover`）无法执行

###  查看日志
```shell
[root@master redis]# tailf /var/log/redis/redis-sentinel.log 
26504:X 24 Apr 2019 16:49:21.876 # Sentinel ID is 77c822dee8e0937f9cfa339b453b71033024ba34
26504:X 24 Apr 2019 16:49:21.876 # +monitor master mymaster 172.18.1.110 6379 quorum 1
26504:X 24 Apr 2019 16:49:22.137 # +new-epoch 5
26504:X 24 Apr 2019 16:49:26.864 # +sdown master mymaster 172.18.1.110 6379
26504:X 24 Apr 2019 16:49:26.864 # +odown master mymaster 172.18.1.110 6379 #quorum 1/1
26504:X 24 Apr 2019 16:49:26.864 # +new-epoch 6
26504:X 24 Apr 2019 16:49:26.864 # +try-failover master mymaster 172.18.1.110 6379
26504:X 24 Apr 2019 16:49:26.870 # +vote-for-leader 77c822dee8e0937f9cfa339b453b71033024ba34 6
26504:X 24 Apr 2019 16:49:26.870 # +sdown slave 10.10.15.199:6379 10.10.15.199 6379 @ mymaster 172.18.1.110 6379
26504:X 24 Apr 2019 16:49:26.870 # +sdown sentinel 8fdcd8ccb9875c9ad43789c2afd1bd105b3e9093 172.18.1.111 26379 @ mymaster 172.18.1.110 6379
26504:X 24 Apr 2019 16:49:37.112 # -failover-abort-not-elected master mymaster 172.18.1.110 6379
26504:X 24 Apr 2019 16:49:37.167 # Next failover delay: I will not start a failover before Wed Apr 24 16:51:27 2019

```

出现该状况原因很多，需要仔细排查日志和各种服务状态。

1. ip `ping` 不通；
2. 防火墙原因；
3. 配置不对，尤其是`protected-mode`配置没有对应上。

## **注意事项**

- 建议详细阅读官方文档相关章节和配置中的注释部分

- **不要**在生产环境中使用双哨兵模式配置

    双机仅为功能演示，实际应用中，请至少使用三机，原因请参考哨兵机制。
    
    如果左边 M1 运行的停止工作，S1 也停止工作。在另一个边 S2 中运行的 Sentinel 将无法授权故障转移，因此系统将无法使用。
    请注意，为了定制不同的故障转移，需要较多数量的哨兵，然后将最新配置传播到所有 Sentinels。另请注意，在没有任何协议的情况下，在上述设置的单个方面进行故障转移非常危险：

    ```shell
    +----+           +------+
    | M1 |----//-----| [M1] |
    | S1 |           | S2   |
    +----+           +------+
    ```
    在上面的配置中，我们以完全对称的方式创建了两个主服务器（假设 S2 可以在未经授权的情况下进行故障转移）。客户端可以无限期地向双方写入，将无法理解分区何时恢复正确的配置，可能导致永久性的裂脑情况。

- 原来的主机切换成从机之后，故障恢复不会主动升主。

    这很符合逻辑，手动`down`掉（原）从机，查看从机哨兵日志
    ```shell
    26661:X 25 Apr 2019 09:51:46.083 # +new-epoch 2
    26661:X 25 Apr 2019 09:51:46.086 # +vote-for-leader 4a818323037cbd9aef1d2d6fd7a2257119308adb 2
    26661:X 25 Apr 2019 09:51:46.086 # +sdown master root 172.18.1.111 6379
    26661:X 25 Apr 2019 09:51:46.086 # +odown master root 172.18.1.111 6379 #quorum 1/1
    26661:X 25 Apr 2019 09:51:46.086 # Next failover delay: I will not start a failover before Thu Apr 25 09:53:46 2019
    26661:X 25 Apr 2019 09:51:47.155 # +config-update-from sentinel 4a818323037cbd9aef1d2d6fd7a2257119308adb 172.18.1.110 26379 @ root 172.18.1.111 6379
    26661:X 25 Apr 2019 09:51:47.155 # +switch-master root 172.18.1.111 6379 172.18.1.110 6379
    26661:X 25 Apr 2019 09:51:47.156 * +slave slave 172.18.1.111:6379 172.18.1.111 6379 @ root 172.18.1.110 6379
    26661:X 25 Apr 2019 09:51:52.181 # +sdown slave 172.18.1.111:6379 172.18.1.111 6379 @ root 172.18.1.110 6379

    ```
    查看（原）主机日志
    ```shell
    10541:M 25 Apr 2019 21:51:46.314 # Setting secondary replication ID to a9abe72647d6924d84f06e12f0bc26c6dc7e057c, valid up to offset: 293604. New replication ID is 13d6b187b44ca6cec6fa87ebba21c10f014232f1
    10541:M 25 Apr 2019 21:51:46.314 * Discarding previously cached master state.
    10541:M 25 Apr 2019 21:51:46.314 * MASTER MODE enabled (user request from 'id=8 addr=172.18.1.110:37008 fd=7 name=sentinel-4a818323-cmd age=1346 idle=0 flags=x db=0 sub=0 psub=0 multi=3 qbuf=140 qbuf-free=32628 obl=36 oll=0 omem=0 events=r cmd=exec')
    10541:M 25 Apr 2019 21:51:46.317 # CONFIG REWRITE executed with success.
    ```
    登录客户端查看状态
    ```shell
    172.18.1.110:6379> info replication
    # Replication  
    role:master     # 我胡汉三又回来啦
    connected_slaves:0
    master_replid:13d6b187b44ca6cec6fa87ebba21c10f014232f1
    master_replid2:a9abe72647d6924d84f06e12f0bc26c6dc7e057c
    master_repl_offset:299701
    second_repl_offset:293604
    repl_backlog_active:1
    repl_backlog_size:1048576
    repl_backlog_first_byte_offset:114343
    repl_backlog_histlen:185359
    ```
 
- 从机启动后日志报错

    ```shell
    26413:S 25 Apr 2019 08:40:31.919 * MASTER <-> REPLICA sync started
    26413:S 25 Apr 2019 08:40:32.924 # Error condition on socket for SYNC: No route to host
    26413:S 25 Apr 2019 08:40:32.930 * Connecting to MASTER 172.18.1.110:6379
    26413:S 25 Apr 2019 08:40:32.931 * MASTER <-> REPLICA sync started
    26413:S 25 Apr 2019 08:40:33.933 # Error condition on socket for SYNC: No route to host
    ```
    首先检查主备是否可以 ping 通，如果可以 ping 通，再检查防火墙设置。
    
    本示例中使用`systemctl stop firewalld`将防火墙关闭（正常操作应配置白名单），之后连接正常。

    ```shell
    26560:S 25 Apr 2019 09:02:12.142 * Connecting to MASTER 172.18.1.110:6379
    26560:S 25 Apr 2019 09:02:12.142 * MASTER <-> REPLICA sync started
    26560:S 25 Apr 2019 09:02:12.143 * Non blocking connect for SYNC fired the event.
    26560:S 25 Apr 2019 09:02:12.149 * Master replied to PING, replication can continue...
    26560:S 25 Apr 2019 09:02:12.156 * Partial resynchronization not possible (no cached master)
    26560:S 25 Apr 2019 09:02:12.160 * Full resync from master: 01145e904f314ebfe466ee74eba57136f6c3d44c:0
    26560:S 25 Apr 2019 09:02:12.227 * MASTER <-> REPLICA sync: receiving 175 bytes from master
    26560:S 25 Apr 2019 09:02:12.227 * MASTER <-> REPLICA sync: Flushing old data
    26560:S 25 Apr 2019 09:02:12.227 * MASTER <-> REPLICA sync: Loading DB in memory
    26560:S 25 Apr 2019 09:02:12.227 * MASTER <-> REPLICA sync: Finished with success
    ```
-  正常状态下，主从服务启动之后，登录哨兵客户端，查看状态应**均为**：
    
    ```shell
    172.18.1.111:26379> info sentinel
    # Sentinel
    sentinel_masters:1
    sentinel_tilt:0
    sentinel_running_scripts:0
    sentinel_scripts_queue_length:0
    sentinel_simulate_failure_flags:0
    master0:name=root,status=ok,address=172.18.1.110:6379,slaves=1,sentinels=2  # 注意该处 status 和哨兵数量 sentinels
    
    ```

## 参考资料
- [官方文档中关于`Sentinel`的说明](https://redis.io/topics/sentinel)
- [Redis 哨兵模式（sentinel）学习总结及部署记录（主从复制、读写分离、主从切换）](https://www.cnblogs.com/kevingrace/p/9004460.html)
- [高可用 Redis 服务架构分析与搭建](https://www.cnblogs.com/xuning/p/8464625.html)
- [深入浅出 Redis-redis 哨兵集群](https://www.cnblogs.com/jaycekon/p/6237562.html)
- [Redis 进阶实践之 Redis 哨兵集群模式](http://www.cnblogs.com/PatrickLiu/p/8444546.html)
- [Redis 哨兵模式](https://www.cnblogs.com/qinghe123/p/9547884.html)
- [Redis 哨兵（sentinel）](https://blog.csdn.net/a67474506/article/details/50435498)