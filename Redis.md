[toc]

# Redis概述

1. `Redis`是一个高性能的（`Key`/`Value`）分布式内存数据库，基于内存运行，并支持持久化的`NoSQL`数据库

2. `Redis`与其他`key-value`缓存产品有以下三个特点：
   - `Redis`支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用
   
   - `Redis`不仅仅支持简单的 `key-value` 类型的数据，同时还提供`list`、`set`、`zset`、`hash`等数据结构的存储
   
   - `Redis`支持数据的备份，即`master-slave`模式的数据备份
   
3. `Redis`支持以下操作：

   - 内存存储和持久化：`Redis`支持异步将内存中的数据写到硬盘上，同时不影响继续服务
   - 取最新`N`个数据的操作，如：可以将最新的10条评论的`ID`放在`Redis`的`List`集合里面
   - 发布、订阅消息系统
   - 地图信息分析
   - 定时器、计数器等等

# Redis下载与安装

## Windows安装

1. 下载地址：https://redis.io/download ，下载安装之后，在电脑的安装目录下：

   ![image-20211231152154352](Redis.assets/image-20211231152154352.png)

2. 启动运行，双击`redis-server.exe`启动即可：

   ![image-20211231152346694](Redis.assets/image-20211231152346694.png)

3. 通过客户端访问

   ```text
   127.0.0.1:6379> set key wyj
   OK
   127.0.0.1:6379> get key
   "wyj"
   127.0.0.1:6379>
   ```

## Linux安装

安装步骤如下：

1. 下载`redis`安装包`redis-6.2.6.tar.gz`（`linux`环境下的安装文件压缩包）：https://redis.io/download ，放在 `/opt`目录下

2. `/opt` 目录下，解压：

   ```bash
   tar -zxvf redis-6.2.6.tar.gz
   ```

3. 解压出现`redis-6.2.6`文件夹，进入目录并执行`make`、`make install`指令

   ```bash
   cd redis-6.2.6
   make
   make install
   ```

   `make install`指令执行完安装之后，会将`Redis`默认安装在`usr/local/bin`下：

   ![image-20220102132839968](Redis.assets/image-20220102132839968.png)

   `/usr`目录，类似于`windows`下的Program Files，用来存放用户的程序

4. 在`/opt/redis-6.2.6`目录下备份`redis.conf`文件

   ```bash
   mkdir myRedisConfigCopy
   cp redis.conf myRedisConfigCopy
   ```

   ![image-20220102133217806](Redis.assets/image-20220102133217806.png)

5. 修改`redis.conf`文件保证可以后台运行

   ```bash
   vim redis.conf
   ```

   ```text
   daemonize no 修改为 daemonize yes
   ```

6. 启动测试：

   ```bash
   # 【shell】启动redis服
   [root@VM-24-11-centos redis-6.2.6]# cd /usr/local/bin
   [root@VM-24-11-centos bin]# ls -a
   .  ..  busybox-x86_64  dump.rdb  redis-benchmark  redis-check-aof  redis-check-rdb  redis-cli  redis-sentinel  redis-server
   [root@VM-24-11-centos bin]# redis-server /opt/redis-6.2.6/redis.conf 
   
   # 【redis客户端连接】 观察地址的变化,redis默认端口号 6379
   [root@VM-24-11-centos bin]# redis-cli -p 6379
   127.0.0.1:6379> ping
   PONG
   127.0.0.1:6379> keys *
   1) "key1"
   127.0.0.1:6379> get key1
   "helloRedis"
   
   # 【redis】关闭连接
   127.0.0.1:6379> shutdown
   not connected> exit
   
   # 【shell】ps显示系统当前进程信息
   [root@VM-24-11-centos bin]# ps -ef|grep redi
   root     26803 19815  0 13:47 pts/0    00:00:00 grep --color=auto redi
   ```

   ![image-20220102134305939](Redis.assets/image-20220102134305939.png)

   ![image-20220102134826476](Redis.assets/image-20220102134826476.png)

# Redis数据类型

## 五大数据类型

`Redis` 支持五种数据类型：`string`（字符串），`list`（列表），`set`（集合），`hash`（哈希）及 `zset`(`sorted set`：有序集合)。`redis` 中的数据都是字符串，由于`redis` 是单线程，不适合存储比较大的数据

1. `string`：存储字符串、整数、浮点数

   ```text
   设置值: set key value
   获取值: get key
   删除键值: del key
   值加一: incr key
   值减一: decr key
   ```

2. `list`：数据结构中的双链表+队列， 可以从左添加元素，也可以从右添加元素

   ```text
   从右边添加元素: rpush listKey value1 value2 ... valueN
   从左边添加元素: lpush listKey value1 value2 ... valueN
   获取指定范围内元素: lrange listKey 0 -1(-1 代表最后一个元素，-2 代表倒数第二个元素，以此类推)
   从左边取值,删除: lpop listKey
   从右边取值,删除: rpop listKey
   ```

3. `set`：无序，元素不能重复，集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 `O(1)`

   ```text
   添加元素: sadd setKey value1 value2 ... valueN
   查询元素: smembers setKey
   删除元素: srem keySet value
   ```

4. `hash`：相当于一个 `key` 对应一个 `map` (`map` 中又是 `key-value`)

   ```text
   设置值: hset hashKey sub-key value
   获取值: hget hashKey sub-key
   获取所有值: hgetall hashKey
   删除值: hdel hashKey sub-key
   ```

5. `zset`：有顺序，不能重复

   ```text
   添加元素: zadd key score1 member1 [score2 member2]
   查看分数:
      1. 从小到大: zrange key 0 -1 [withscores]
      2. 从大到小: zrevrange key 0 -1 [withscores]
   对元素 member 增加 score: zincrby key score member
   ```

另外关于`key`键的操作：

```
查看所有的key: keys * 
判断某个key是否存在: exists keyName
将当前数据库的key移动到给定的数据库db当中: move key db
为给定key设置生存时间,当key过期时(生存时间为0),它会被自动删除: expire key 秒钟
查看还有多少秒过期,-1表示永不过期,-2表示已过期: ttl key 
查看key的类型: type key 
```

## 三种特殊类型

### GEO地理位置

### HyperLogLog

### BitMap

# Redis.config

`Redis.config`位于`Redis`的安装目录下，建议备份来保证初始文件的安全：

![image-20220102140846734](Redis.assets/image-20220102140846734.png)

部分配置文件内容如下：

1. 配置大小单位，开头定义了一些基本的度量单位，只支持bytes，不支持bit

   ```
   # Note on units: when memory size is needed, it is possible to specify
   # it in the usual form of 1k 5GB 4M and so forth:
   #
   # 1k => 1000 bytes
   # 1kb => 1024 bytes
   # 1m => 1000000 bytes
   # 1mb => 1024*1024 bytes
   # 1g => 1000000000 bytes
   # 1gb => 1024*1024*1024 bytes
   #
   # units are case insensitive so 1GB 1Gb 1gB are all the same.
   
   ```

2. `network`网络配置

   ```bash
   bind 127.0.0.1 -::1 # 绑定的ip
   protected-mode yes  # 保护模式
   port 6379           # 默认端口
   ```

3. 通用

   ```bash
   daemonize yes  # 默认情况下, Redis 不作为守护进程运行, 需要开启的话改为 yes
   
   supervised no  # 可通过 upstart 和 systemd 管理 Redis 守护进程
   
   pidfile /var/run/redis_6379.pid  # 以后台进程方式运行 Redis, 则需要指定 pid 文件
   
   loglevel notice # 日志级别
   	可选项有：# debug(记录大量日志信息，适用于开发、测试阶段)
   			# verbose(较多日志信息)
   			# notice(适量日志信息，使用于生产环境)
   			# warning(仅有部分重要、关键信息才会被记录)
   logfile ""  # 日志文件的位置, 当指定为空字符串时, 为标准输出
   
   databases 16# 设置数据库的数目, 默认的数据库是 DB 0
   			# redis 共有 16 个数据库, 分别使用 0-15 索引得到
   			
   always-show-logo yes # 是否总是显示 logo	
   ```

4. `Snapshot` 快照

   ```bash
   # 900秒(15分钟)内至少1个 key 值改变(则进行数据库保存--持久化)
   save 900 1
   # 300秒(5分钟)内至少10个 key 值改变(则进行数据库保存--持久化)
   save 300 10
   # 60秒(1分钟)内至少10000个 key 值改变(则进行数据库保存--持久化)
   save 60 10000
   
   stop-writes-on-bgsave-error yes
   # 持久化出现错误后, 是否依然进行继续进行工作
   
   rdbcompression yes
   # 使用压缩 rdb 文件 
   # yes: 压缩, 但是需要一些 CPU 的消耗
   # no: 不压缩, 需要更多的磁盘空间
   
   rdbchecksum yes 
   # 是否校验 rdb 文件, 更有利于文件的容错性, 但是在保存 rdb 文件的时候, 会有大概10%的性能损耗
   
   dbfilename dump.rdb  # dbfilenamerdb文件名称
   
   dir ./    
   # dir 数据目录，数据库的写入会在这个目录。rdb、aof文件也会写在这个目录
   ```

配置文件内容有很多，此处不一一列举

## Redis命令

远程连接`Redis`命令：

```bash
redis-cli -h host -p port -a password
```

# Redis持久化方案

`Redis` 支持两种持久化策略：`RDB` 快照和 `AOF` 日志，而 `Memcached` 不支持持久化。`redis` 默认开启 `RDB`，同时开启两个持久化方案，则按照 `AOF` 的持久化放案恢复数据。

## RDB快照

### RDB快照简介

1. 定期将当前时刻的数据保存磁盘中，`Redis`会单独创建（`fork`）一个子进程来进行持久化，会产生一个 `dump.rdb` 文件，默认存放在`/user/local/bin`目录下

   ![image-20220102145459774](Redis.assets/image-20220102145459774.png)

2. `fork`的作用是复制一个与当前进程一样的进程，新进程的所有数据（变量，环境变量，程序计数器等）数值都和原进程一致，并作为原进程的子进程。先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的`dump.rdb` 文件

3. 优缺点：

   | 优点                         | 缺点                                                         |
   | :--------------------------- | :----------------------------------------------------------- |
   | 适合大规模的数据恢复         | 在一定间隔时间做一次备份，所以如果`Redis`意外`down`掉的话，就会丢失最后一次快照后的所有修改 |
   | 对数据完整性和一致性要求不高 | `Fork`的时候，内存中的数据被克隆了一份，大致 2 倍的膨胀性需要考虑 |

###  触发、关闭、恢复

1. **手动触发**，调用 `save`，`bgsave` 命令
   
   - `save` 命令会阻塞`Redis`服务器进程，直到`RDB`文件创建完毕为止，在`Redis`服务器阻塞期间，服务器不能处理任何命令请求
   - `bgsave` 命令会创建一个子进程，由子进程来负责创建`RDB`文件，父进程(即`Redis`主进程)则继续处理请求
   
2. **自动触发**，最常见的情况是在配置文件中通过 `save m n` ，指定当`m`秒内发生`n`次变化时，会触发 `bgsave`。`Redis`的默认配置文件(`Linux`下为`Redis`根目录下的`redis.conf`)，可以看到如下配置信息：

   ```text
   save 900 1
   save 300 10
   save 60 10000
   ```

3. **关闭方式**：如果想禁用`RDB`持久化的策略，只要不设置任何`save`指令，或者给`save`传入一个空字符串参数也可以

4. **日志恢复**：将备份文件（`dump.rdb`）移动到`Redis`安装目录并启动服务即可

## AOF日志

1. `RDB` 持久化是将进程数据写入文件，而 `AOF` 持久化(即 `Append Only File` 持久化)，则是将 `Redis` 执行的每次写命令记录到单独的日志文件中（有点像 `MySQL` 的 `binlog`）；当 `Redis` 重启时再次执行 `AOF`  文件中的命令来恢复数据。特点是每秒保存，数据比较完整，耗费性能。

2. **`AOF` 开启设置**：修改 `redis.conf` 文件，将 `appendonly` 设置为 `yes`，默认关闭，数据存在 `appendonly.aof` 文件中

3. 配置文件`redis.cofig`中关于`AOF`日志的内容如下：

   ```bash
   # 是否以 append only 模式作为持久化方式, 默认使用的是rdb方式持久化, 该方式在许多应用中已经足够用了
   appendonly no   
   
   # appendfilename AOF 文件名称
   appendfilename "appendonly.aof"
   
   appendfsync everysec    # appendfsync aof 持久化策略的配置
   						# no 表示不执行 fsync, 由操作系统保证数据同步到磁盘, 速度最快。
   						# always 表示每次写入都执行 fsync, 以保证数据同步到磁盘
   						# everysec 表示每秒执行一次 fsync, 可能会导致丢失这 1s 数据。
   
   No-appendfsync-on-rewrite # 重写时是否可以运用 Appendfsync, 用默认 no 即可, 保证数据安全性
   Auto-aof-rewrite-min-size # 设置重写的基准值
   Auto-aof-rewrite-percentage # 设置重写的基准值
   ```

4. 优缺点：

   | 优点                                                         | 缺点                                                         |
   | ------------------------------------------------------------ | ------------------------------------------------------------ |
   | `appendfsync always` ，每次发生数据变更会被立即记录到磁盘，性能较差但数据完整性比较好 | 相同数据集的数据而言，`AOF` 文件要远大于 `rdb`文件，恢复速度慢于 `RDB` |
   | `appendfsync everysec` 异步操作，如果一秒内宕机，有数据丢失  | `AOF` 运行效率要慢于 `RDB`，每秒同步策略效率较好，不同步效率和`RDB`相同 |
   | 不同步： `appendfsync no`  从不同步                          |                                                              |

# 使用Jedis操作Redis

## Jedis远程连接云服务器Redis

`Jedis`远程连接`Linux`云服务器下的`Redis`，需要在`Linux`服务器完成以下操作：

1. 使用`Xshell`工具连接到远程服务器，进入`Redis`安装目录，找到并修改`redis.conf`文件夹配置内容

   ```bash
   vim redis.conf
   ```

   ```bash
   # Jedis 连接 云服务器 Redis 修改配置内容
   daemonize yes 
   # protected-mode 配置了外网保护, 默认是yes
   protected-mode no 
   注释掉 bind 127.0.0.1
   ```

2. 启动`Redis`服务

   ```bash
   [root@VM-24-11-centos redis-6.2.6]# cd /usr/local/bin
   [root@VM-24-11-centos bin]# redis-server /opt/redis-6.2.6/redis.conf 
   [root@VM-24-11-centos bin]# redis-cli -p 6379
   127.0.0.1:6379> ping
   PONG
   ```

   ![image-20220102221207316](Redis.assets/image-20220102221207316.png)

3. 配置防火墙/安全组，开启 `6379` 端口号（`Redis`的默认端口号）

   腾讯云轻量级云服务器配置方式如图所示：

   ![image-20220102221655166](Redis.assets/image-20220102221655166.png)

之后便可以远程连接云服务器`Redis`

## Jedis连接Redis代码详解

配置好云服务器的`Redis`之后，便可以使用`Jedis`连接远程云服务器的`Redis`，具体代码如下：

1. 新建`Maven`项目`redisproject`，在项目中新建名为`jedistest`的`Module`，导入`Jedis`依赖包

   ```xml
   <dependency>
       <groupId>redis.clients</groupId>
       <artifactId>jedis</artifactId>
       <version>3.2.0</version>
   </dependency>
   <!-- https://mvnrepository.com/artifact/com.alibaba/fastjson -->
   <dependency>
       <groupId>com.alibaba</groupId>
       <artifactId>fastjson</artifactId>
       <version>1.2.58</version>
   </dependency>
   ```

2. 开启`Linunx`服务器端`Redis`

   ![image-20220103091429648](Redis.assets/image-20220103091429648.png)

3. 编写测试代码并测试

   ```java
   public class JedisTest {
       @Test
       public void jedis() {
           Jedis jedis = new Jedis("101.43.164.126",6379);
           System.out.println("连接成功");
           // 查看服务是否运行
           System.out.println("服务正在运行: "+ jedis.ping() );
       }
   }
   ```

   测试结果：

   ![image-20220103125841768](Redis.assets/image-20220103125841768.png)

Jedis常用API参考：<https://www.pianshen.com/article/2355326280/>

RedisTemplate常用方法参考：

<https://blog.csdn.net/sinat_22797429/article/details/89196933>

# Redis主从复制

### 主从复制简介

- 主从复制是指将一台`Redis`服务器的数据，复制到其他的`Redis`服务器。前者称为主节点（`Master`/`Leader`），后者称为从节点（`Slave`/`Follower`）
- 主从复制中数据的复制是单向的，只能由主节点复制到从节点，主节点以写为主，从节点以读为主
- 默认情况下，每台`Redis`服务器都是主节点，一个主节点可以有 0 个或者多个从节点，但每个从节点只能有一个主节点。

启动`Redis`服务通过`info replication`观察`Redis`服务器状态信息：

```bash
127.0.0.1:6379> info replication
# Replication
role:master # 角色
connected_slaves:0 # 从机数量
master_failover_state:no-failover
master_replid:57d795b7d20ed8bd9796980c12083fcb5527cc34
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
127.0.0.1:6379> 
```

![image-20220104111606136](Redis.assets/image-20220104111606136.png)

### 主从复制配置流程

配置单机多服务集群方式如下，以一主机二从机（共3个服务器）举例：

1. 在`Redis`解压目录下创建存放配置文件的文件夹`config`，在该文件夹将原来的redis.config文件拷贝三份并命名为`redis6379.conf`、`redis6380.conf`、`redis6381.conf`

   ```bash
   [root@localserver-1 redis-6.2.6]# cp redis.conf config/redis6380.conf
   [root@localserver-1 redis-6.2.6]# cp redis.conf config/redis6381.conf
   [root@localserver-1 redis-6.2.6]# cp redis.conf config/redis6379.conf
   ```

2. 修改`redis6379.conf`、`redis6380.conf`、`redis6381.conf`内容，每个配置文件对应修改如下内容：

   - port：端口号
   - pidfile：pid文件名
   - logfile：日志存储位置
   - dbfilename：数据库文件名

   以`redis6380.conf`为例：

   ```bash
   [root@localserver-1 config]# vim redis6380.conf 
   
   port 6380
   dbfilename dump6380.rdb 
   logfile /var/log/redisLog/redis6380.log # 在 /var/log/redisLog 中存 Redis 日志信息
   pidfile /var/run/redis_6380.pid
   ```

3. 设置一主（79）二从（80、81）服务器。只设置从服务器即可，使用`SLAVEOF host port`就可以为从机配置主机了。

   ```bash
   # 启动端口号为 6379 的服务器
   [root@localserver-1 bin]# redis-server /opt/redis-6.2.6/config/redis6379.conf 
   [root@localserver-1 bin]# redis-cli -p 6379
   127.0.0.1:6379> ping
   PONG
   127.0.0.1:6379> 
   
   # 启动端口号为 6380 的服务器, 并配置其主服务器为 6379
   [root@localserver-1 wyj]# cd /usr/local/bin
   [root@localserver-1 bin]# redis-server /opt/redis-6.2.6/config/redis6380.conf 
   [root@localserver-1 bin]# redis-cli -p 6380
   127.0.0.1:6380> ping
   PONG
   127.0.0.1:6380> 
   127.0.0.1:6380> SLAVEOF 127.0.0.1 6379
   OK
   127.0.0.1:6380> 
   
   # 启动端口号为 6381 的服务器, 并配置其主服务器为 6379
   [root@localserver-1 wyj]# cd /usr/local/bin
   [root@localserver-1 bin]# redis-server /opt/redis-6.2.6/config/redis6381.conf 
   [root@localserver-1 bin]# redis-cli -p 6381
   127.0.0.1:6381> ping
   PONG
   127.0.0.1:6381> 
   127.0.0.1:6381> SLAVEOF 127.0.0.1 6379
   OK
   127.0.0.1:6381> 
   
   
   # 查看 Redis 进程信息
   [wyj@localserver-1 ~]$ ps -ef | grep redis
   wyj         3120    2156  0 11:21 ?        00:00:12 redis-server 127.0.0.1:6379
   root        4805    2156  0 12:51 ?        00:00:00 redis-server 127.0.0.1:6380
   root        4819    4750  0 12:51 pts/1    00:00:00 redis-cli -p 6380
   root        4935    2156  0 12:52 ?        00:00:00 redis-server 127.0.0.1:6381
   root        4946    4881  0 12:52 pts/2    00:00:00 redis-cli -p 6381
   root        4996    4531  0 12:55 pts/0    00:00:00 redis-cli -p 6379
   wyj         5050    5013  0 12:56 pts/3    00:00:00 grep --color=auto redis
   [wyj@localserver-1 ~]$ 
   ```

4. 使用`info replication`命令查看配置是否成功

   ```bash
   # 主服务器显示有两个从服务器 connected_slaves:2
   127.0.0.1:6379> info replication
   # Replication
   role:master
   connected_slaves:2
   slave0:ip=127.0.0.1,port=6380,state=online,offset=112,lag=1
   slave1:ip=127.0.0.1,port=6381,state=online,offset=112,lag=1
   master_failover_state:no-failover
   master_replid:04e13bd6174e645708bde344ff937d940fa69628
   master_replid2:0000000000000000000000000000000000000000
   master_repl_offset:112
   second_repl_offset:-1
   repl_backlog_active:1
   repl_backlog_size:1048576
   repl_backlog_first_byte_offset:1
   repl_backlog_histlen:112
   127.0.0.1:6379>
   
   # 从服务器 6380 显示配置成功, master_xxx 为主服务器信息
   127.0.0.1:6380> info replication
   # Replication
   role:slave
   master_host:127.0.0.1
   master_port:6379
   master_link_status:up
   master_last_io_seconds_ago:8
   master_sync_in_progress:0
   slave_read_repl_offset:154
   slave_repl_offset:154
   slave_priority:100
   slave_read_only:1
   replica_announced:1
   connected_slaves:0
   master_failover_state:no-failover
   master_replid:04e13bd6174e645708bde344ff937d940fa69628
   master_replid2:0000000000000000000000000000000000000000
   master_repl_offset:154
   second_repl_offset:-1
   repl_backlog_active:1
   repl_backlog_size:1048576
   repl_backlog_first_byte_offset:1
   repl_backlog_histlen:154
   127.0.0.1:6380> 
   ```

   ![image-20220104130209218](Redis.assets/image-20220104130209218.png)

### 主从服务器使用规则

1. 从机只能读，不能写，主机可读可写但是多用于写。

   ```bash
   127.0.0.1:6381> set name sakura # 从机 6381 写入失败
   (error) READONLY You can't write against a read only replica.
   
   127.0.0.1:6380> set name sakura # 从机 6380 写入失败
   (error) READONLY You can't write against a read only replica.
   
   127.0.0.1:6379> set name sakura
   OK
   127.0.0.1:6379> get name
   "sakura"
   ```

2. 主机断电宕机后，默认情况下从机的角色不会发生变化 ，集群中只是失去了写操作，主机恢复之后会连接上从机恢复原状

3. 当从机断电宕机后，若不是使用配置文件配置的从机，再次启动后作为主机是无法获取之前主机的数据的，若此时重新配置为从机，又可以获取到主机的所有数据

4. 第二条中提到，默认情况下，主机故障后不会出现新的主机，有两种方式可以产生新的主机：

   - 从机手动执行命令`slaveof no one`，执行后从机会独立出来成为一个主机
   - 使用哨兵模式（自动选举）

# Redis常见错误

## shutdown命令失败

`Linux`服务器端使用`shutdown`命令，出现错误：

```text
ERR Errors trying to SHUTDOWN. Check logs.
```

**解决方案：**

查看Redis存放日志，发现端口被占用：

```tex
26855:M 03 Jan 2022 09:51:03.781 # Warning: Could not create server TCP listening socket *:6379: bind: Address already in use
26855:M 03 Jan 2022 09:51:03.781 # Failed listening on port 6379 (TCP), aborting.
```

查看`Redis`进程并杀死该进程，之后重启`Redis`：

```bash
[root@VM-24-11-centos ~]# ps -ef | grep redis
root      1899     1  0 Jan02 ?        00:00:42 redis-server *:6379
root     17256 16160  0 09:56 pts/2    00:00:00 grep --color=auto redis
root     26881 16155  0 09:51 pts/0    00:00:00 redis-cli -p 6379
root     30113 13108  0 09:52 pts/1    00:00:00 vim redisLog79.log
[root@VM-24-11-centos ~]# kill -9 1899
```

## Jedis连接Redis失败

修改`redis.conf`文件中的以下内容：

```tex
daemonize yes 
protected-mode no 
注释掉 bind 127.0.0.1
```

`Windows`客户端使用`Jedis`连接腾讯云`Linux`服务器`Redis`，第一次连接成功，之后连接失败尝试以下方式：

```bash
# 开启防火墙
systemctl start firewalld
# 重启防火墙
firewall-cmd --reload
# 关闭防火墙
systemctl stop firewalld
```

## 主从配置失败

配置好主从服务器后，从服务使用`info replication`命令，master_link_status`显示为master_link_status:down`没有和主机连上，可以查看从机日志信息

```log
Error reply to PING from master: ‘-MISCONF Redis is configured to save RDB snapshots
```

原因可能是`Redis`使用的内存超过操作系统一半的内存。`Redis`在保存数据到硬盘时为了避免主进程假死，需要`Fork`一份主进程，然后在`Fork`进程内完成数据保存到硬盘的操作，如果主进程使用了`4GB`的内存，`Fork`子进程的时候需要额外的`4GB`，此时内存如果不够，`Fork`则会失败，进而数据保存硬盘指令也会失败。

讲解方式，修改主从服务器的配置文件`stop-writes-on-bgsave-error`为`no`：

```
stop-writes-on-bgsave-error no
```







