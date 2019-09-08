# redis的操作

### 操作String

```
        redisTemplate.opsForValue().set("a", user);
        redisTemplate.opsForValue().set("b", user);
        log.info("取出数据的数据为 {}", redisTemplate.opsForValue().get("a"));

```

### 操作hash

```
        //添加
        redisTemplate.opsForHash().put("spring-boot-hash", "name", "wql");
        redisTemplate.opsForHash().put("spring-boot-hash", "password", "123");
        //取数据
        System.out.println(redisTemplate.opsForHash().get("spring-boot-hash", "name"));
        //删除
        redisTemplate.opsForHash().delete("spring-boot-hash", "name");


        //添加
        HashMap<String, String> map = new HashMap<>(4);
        map.put("name", "cooper");
        map.put("password", "123");
        redisTemplate.opsForHash().putAll("map", map);

        HashMap map1 = (HashMap) redisTemplate.opsForHash().entries("map");
        System.out.println("name="+map1.get("name"));
```

### 操作list

```
        redisTemplate.opsForList().rightPush("list1","第一条数据");
        redisTemplate.opsForList().leftPush("list1","第二条数据");
        //移除左边元素
        System.out.println( redisTemplate.opsForList().leftPop("list1"));
```

### 操作zset

```
        redisTemplate.opsForZSet().add("key", "value1", 1.0);
        redisTemplate.opsForZSet().add("key", "value2", 5.0);
        redisTemplate.opsForZSet().add("key", "value3", 6.0);
        redisTemplate.opsForZSet().add("key", "value4", 4.0);
        redisTemplate.opsForZSet().add("key", "value5", 5.0);
        redisTemplate.opsForZSet().add("key", "value6", 6.0);
        redisTemplate.opsForZSet().add("key", "value7", 7.0);
        redisTemplate.opsForZSet().add("key", "value8", 8.0);
        
        //修改score 实际应用中score可以是积分如类的数据
        redisTemplate.opsForZSet().incrementScore("key", "value1", 9.0);

        //可以做排行榜
        System.out.println(redisTemplate.opsForZSet().rangeByScore("key", 5.0, 10.0));
        
         //判断value在zset中的排名  zrank
        System.out.println("排名是"+redisTemplate.opsForZSet().rank("key", "value3"));
        
        
         //查询value对应的score 
        redisTemplate.opsForZSet().score("key", "value");
```

# 高并发redis问题

### 缓存一致性

- 更新数据库成功，更新缓存失败。
- 更新缓存成功，更新数据库失败。
- 更新数据库成功，删除缓存失败。
- 淘汰缓存成功，更新数据库失败

### 缓存穿透

 高并发场景下，某一个key被高并发访问而缓存中没有，从而大量请求发送到数据库，如一些恶意攻击请求一些不存在的key。

#### 解决方案

- 缓存key，但数据为空。
- 对所有可能对应数据为空的key进行过滤（布隆过滤器），

### 缓存雪崩

由于缓存原因，导致大量请求发送到数据库。原因：缓存系统宕机。

# 持久化

## RDB

RDB 是 Redis 默认的持久化方案。在指定的时间间隔内，执行指定次数的写操作，则会将内存中的数据写入到磁盘中。即在指定目录下生成一个dump.rdb文件。Redis 重启会通过加载dump.rdb文件恢复数据

### 从配置文件了解RDB

打开 redis.conf 文件，找到 SNAPSHOTTING 对应内容
1、 RDB核心规则配置（重点）

```
save <seconds> <changes>
# save ""
save 900 1
save 300 10
save 60 10000
```

解说：save <指定时间间隔> <执行指定次数更新操作>，满足条件就将内存中的数据同步到硬盘中。官方出厂配置默认是 900秒内有1个更改，300秒内有10个更改以及60秒内有10000个更改，则将内存中的数据快照写入磁盘。
若不想用RDB方案，可以把 save "" 的注释打开，下面三个注释。

2 指定本地数据库文件名，一般采用默认的 dump.rdb

```
dbfilename dump.rdb
```

3 指定本地数据库存放目录，一般也用默认配置

```
dir ./
```

4 默认开启数据压缩

```
rdbcompression yes
```

解说：配置存储至本地数据库时是否压缩数据，默认为yes。Redis采用LZF压缩方式，但占用了一点CPU的时间。若关闭该选项，但会导致数据库文件变的巨大。建议开启。

### 触发RDB快照

1 在指定的时间间隔内，执行指定次数的写操作
 2 执行save（阻塞， 只管保存快照，其他的等待） 或者是bgsave （异步）命令
 3 执行flushall 命令，清空数据库所有数据，意义不大。
 4 执行shutdown 命令，保证服务器正常关闭且不丢失任何数据，意义...也不大。

### 通过RDB文件恢复数据

将dump.rdb 文件拷贝到redis的安装目录的bin目录下，重启redis服务即可。在实际开发中，一般会考虑到物理机硬盘损坏情况，选择备份dump.rdb 。可以从下面的操作演示中可以体会到。

### RDB 的优缺点

优点：
 1 适合大规模的数据恢复。
 2 如果业务对数据完整性和一致性要求不高，RDB是很好的选择。

缺点：
 1 数据的完整性和一致性不高，因为RDB可能在最后一次备份时宕机了。
 2 备份时占用内存，因为Redis 在备份时会独立创建一个子进程，将数据写入到一个临时文件（此时内存中的数据是原来的两倍哦），最后再将临时文件替换之前的备份文件。
 所以Redis 的持久化和数据的恢复要选择在夜深人静的时候执行是比较合理的。

### 操作演示

```
[root@itdragon bin]# vim redis.conf
save 900 1
save 120 5
save 60 10000
[root@itdragon bin]# ./redis-server redis.conf
[root@itdragon bin]# ./redis-cli -h 127.0.0.1 -p 6379
127.0.0.1:6379> keys *
(empty list or set)
127.0.0.1:6379> set key1 value1
OK
127.0.0.1:6379> set key2 value2
OK
127.0.0.1:6379> set key3 value3
OK
127.0.0.1:6379> set key4 value4
OK
127.0.0.1:6379> set key5 value5
OK
127.0.0.1:6379> set key6 value6
OK
127.0.0.1:6379> SHUTDOWN
not connected> QUIT
[root@itdragon bin]# cp dump.rdb dump_bk.rdb
[root@itdragon bin]# ./redis-server redis.conf
[root@itdragon bin]# ./redis-cli -h 127.0.0.1 -p 6379
127.0.0.1:6379> FLUSHALL 
OK
127.0.0.1:6379> keys *
(empty list or set)
127.0.0.1:6379> SHUTDOWN
not connected> QUIT
[root@itdragon bin]# cp dump_bk.rdb  dump.rdb
cp: overwrite `dump.rdb'? y
[root@itdragon bin]# ./redis-server redis.conf
[root@itdragon bin]# ./redis-cli -h 127.0.0.1 -p 6379
127.0.0.1:6379> keys *
1) "key5"
2) "key1"
3) "key3"
4) "key4"
5) "key6"
6) "key2"
```

第一步：vim 修改持久化配置时间，120秒内修改5次则持久化一次。
第二步：重启服务使配置生效。
第三步：分别set 5个key，过两分钟后，在bin的当前目录下会自动生产一个dump.rdb文件。（set key6 是为了验证shutdown有触发RDB快照的作用）
第四步：将当前的dump.rdb 备份一份（模拟线上工作）。
第五步：执行FLUSHALL命令清空数据库数据（模拟数据丢失）。
第六步：重启Redis服务，恢复数据.....咦？？？？( ′◔ ‸◔`)。数据是空的？？？？这是因为FLUSHALL也有触发RDB快照的功能。
第七步：将备份的 dump_bk.rdb 替换 dump.rdb 然后重新Redis。

注意点：SHUTDOWN 和 FLUSHALL 命令都会触发RDB快照，这是一个坑，请大家注意。

其他命令：

- keys * 匹配数据库中所有 key
- save 阻塞触发RDB快照，使其备份数据
- FLUSHALL 清空整个 Redis 服务器的数据(几乎不用)
- SHUTDOWN 关机走人（很少用）

## AOF

AOF ：Redis 默认不开启。它的出现是为了弥补RDB的不足（数据的不一致性），所以它采用日志的形式来记录每个**写操作**，并**追加**到文件中。Redis 重启的会根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作。

### 从配置文件了解AOF

打开 redis.conf 文件，找到 APPEND ONLY MODE 对应内容
1 、redis 默认关闭，开启需要手动把no改为yes

```
appendonly yes
```

2 、指定本地数据库文件名，默认值为 appendonly.aof

```
appendfilename "appendonly.aof"
```

3 、指定更新日志条件

```
# appendfsync always
appendfsync everysec
# appendfsync no
```

解说：
 always：同步持久化，每次发生数据变化会立刻写入到磁盘中。性能较差当数据完整性比较好（慢，安全）
 everysec：出厂默认推荐，每秒异步记录一次（默认值）
 no：不同步

4 配置重写触发机制

```
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```

解说：当AOF文件大小是上次rewrite后大小的一倍且文件大于64M时触发。一般都设置为3G，64M太小了。

### 触发AOF快照

根据配置文件触发，可以是每次执行触发，可以是每秒触发，可以不同步。

### 根据AOF文件恢复数据

正常情况下，将appendonly.aof 文件拷贝到redis的安装目录的bin目录下，重启redis服务即可。但在实际开发中，可能因为某些原因导致appendonly.aof 文件格式异常，从而导致数据还原失败，可以通过命令redis-check-aof --fix appendonly.aof 进行修复 。从下面的操作演示中体会。

### AOF的重写机制

前面也说到了，AOF的工作原理是将写操作追加到文件中，文件的冗余内容会越来越多。所以聪明的 Redis 新增了重写机制。当AOF文件的大小超过所设定的阈值时，Redis就会对AOF文件的内容压缩。

重写的原理：Redis 会fork出一条新进程，读取内存中的数据，并重新写到一个临时文件中。并没有读取旧文件（你都那么大了，我还去读你？？？ o(ﾟДﾟ)っ傻啊！）。最后替换旧的aof文件。

触发机制：当AOF文件大小是上次rewrite后大小的一倍且文件大于64M时触发。这里的“一倍”和“64M” 可以通过配置文件修改。

### AOF 的优缺点

优点：数据的完整性和一致性更高
缺点：因为AOF记录的内容多，文件会越来越大，数据恢复也会越来越慢。

### 操作演示

```
[root@itdragon bin]# vim appendonly.aof
appendonly yes
[root@itdragon bin]# ./redis-server redis.conf
[root@itdragon bin]# ./redis-cli -h 127.0.0.1 -p 6379
127.0.0.1:6379> keys *
(empty list or set)
127.0.0.1:6379> set keyAOf valueAof
OK
127.0.0.1:6379> FLUSHALL 
OK
127.0.0.1:6379> SHUTDOWN
not connected> QUIT
[root@itdragon bin]# ./redis-server redis.conf
[root@itdragon bin]# ./redis-cli -h 127.0.0.1 -p 6379
127.0.0.1:6379> keys *
1) "keyAOf"
127.0.0.1:6379> SHUTDOWN
not connected> QUIT
[root@itdragon bin]# vim appendonly.aof
fjewofjwojfoewifjowejfwf
[root@itdragon bin]# ./redis-server redis.conf
[root@itdragon bin]# ./redis-cli -h 127.0.0.1 -p 6379
Could not connect to Redis at 127.0.0.1:6379: Connection refused
not connected> QUIT
[root@itdragon bin]# redis-check-aof --fix appendonly.aof 
'x              3e: Expected prefix '*', got: '
AOF analyzed: size=92, ok_up_to=62, diff=30
This will shrink the AOF from 92 bytes, with 30 bytes, to 62 bytes
Continue? [y/N]: y
Successfully truncated AOF
[root@itdragon bin]# ./redis-server redis.conf
[root@itdragon bin]# ./redis-cli -h 127.0.0.1 -p 6379
127.0.0.1:6379> keys *
1) "keyAOf"
```

第一步：修改配置文件，开启AOF持久化配置。
第二步：重启Redis服务，并进入Redis 自带的客户端中。
第三步：保存值，然后模拟数据丢失，关闭Redis服务。
第四步：重启服务，发现数据恢复了。（额外提一点：有教程显示FLUSHALL 命令会被写入AOF文件中，导致数据恢复失败。我安装的是redis-4.0.2没有遇到这个问题）。
第五步：修改appendonly.aof，模拟文件异常情况。
第六步：重启 Redis 服务失败。这同时也说明了，RDB和AOF可以同时存在，且优先加载AOF文件。
第七步：校验appendonly.aof 文件。重启Redis 服务后正常。

补充点：aof 的校验是通过 redis-check-aof 文件，那么rdb 的校验是不是可以通过 redis-check-rdb 文件呢？？？

### 总结

1. Redis 默认开启RDB持久化方式，在指定的时间间隔内，执行指定次数的写操作，则将内存中的数据写入到磁盘中。
2. RDB 持久化适合大规模的数据恢复但它的数据一致性和完整性较差。
3. Redis 需要手动开启AOF持久化方式，默认是每秒将写操作日志追加到AOF文件中。
   
4. AOF 的数据完整性比RDB高，但记录内容多了，会影响数据恢复的效率。
5. Redis 针对 AOF文件大的问题，提供重写的瘦身机制。
6. 若只打算用Redis 做缓存，可以关闭持久化。
7. 若打算使用Redis 的持久化。建议RDB和AOF都开启。其实RDB更适合做数据的备份，留一后手。AOF出问题了，还有RDB。

 # 集群

## 主从复制

主从复制，是指将一台Redis服务器的数据，复制到其他的Redis服务器。前者称为主节点(master)，后者称为从节点(slave)；数据的复制是单向的，只能由主节点到从节点。

1、配置文件

在从服务器的配置文件中加入：slaveof <masterip> <masterport>

2、启动命令

redis-server启动命令后加入 --slaveof <masterip> <masterport>

3、客户端命令

Redis服务器启动后，直接通过客户端执行命令：slaveof <masterip> <masterport>，则该Redis实例成为从节点。

#### 全量同步

Redis全量复制一般发生在Slave初始化阶段，这时Slave需要将Master上的所有数据都复制一份。具体步骤如下： 

-  从服务器连接主服务器，发送SYNC命令； 
-  主服务器接收到SYNC命名后，开始执行BGSAVE命令生成RDB文件并使用缓冲区记录此后执行的所有写命令； 
-  主服务器BGSAVE执行完后，向所有从服务器发送快照文件，并在发送期间继续记录被执行的写命令； 
-  从服务器收到快照文件后丢弃所有旧数据，载入收到的快照； 
-  主服务器快照发送完毕后开始向从服务器发送缓冲区中的写命令； 
-  从服务器完成对快照的载入，开始接收命令请求，并执行来自主服务器缓冲区的写命令；

#### 增量同步

- 如果网络发生抖动，2.8版本之前会重新生产rdb文件，然后重新执行
- 之后，提供部分复制功能、如果抖动，master会在复制缓冲区生成一个buffer（默认1M），如果offset在buffer范围内，则会将部分数据复制到slave，而不需要全量复制

1. redis什么时候会发生全量复制？

   a) redis slave首启动或者重启后，连接到master时

   b) redis slave进程没重启，但是掉线了，重连后不满足部分复制条件

2. redis什么时候会发生部分复制？

   部分复制需要的条件

   a) 主从的redis版本>=2.8

   b) redis slave进程没有重启，但是掉线了，重连了master(因为slave进程重启的话，run id就没有了)

   c) redis slave保存的run id与master当前run id一致 (注：run id并不是pid，slave把它保存在内存中，重启就消失)

   d) redis slave掉线期间，master保存在内存的offset可用，也就是master变化不大，被更改的指令都保存在内存

3. redis进程重启后会发生全量复制还是部分复制？

  a) master重启时，run id会发生变化

  b) slave重启时，run id会丢失

会发生全量复制，因为部分复制的条件之一run id已经不能满足

**缺点：**

没有实现故障自动转移，哨兵模式实现故障自动转移

## 哨兵模式

### 概念

Redis-Sentinel是Redis官方推荐的高可用性(HA)解决方案。实际上这意味着你可以使用Sentinel模式创建一个可以不用人为干预而应对各种故障的Redis部署。

- **监控(Monitoring)**: Sentinel 会不断地定期检查你的主服务器和从服务器是否运作正常。
- **提醒(Notification)**: 当被监控的某个 Redis 服务器出现问题时， Sentinel 可以通过 API 向管理员或者其他应用程序发送通知。
- 它的主要功能有以下几点
  - 监控：Sentinel不断的检查master和slave是否正常的运行。
  - 通知：如果发现某个redis节点运行出现问题，可以通过API通知系统管理员和其他的应用程序。
  - 自动故障转移：能够进行自动切换。当一个master节点不可用时，能够选举出master的多个slave中的一个来作为新的master,其它的slave节点会将它所追随的master的地址改为被提升为master的slave的新地址。
  - 配置提供者：哨兵作为Redis客户端发现的权威来源：客户端连接到哨兵请求当前可靠的master的地址。如果发生故障，哨兵将报告新地址。
- **自动故障迁移(Automaticfailover)**: 当一个主服务器不能正常工作时， Sentinel 会开始一次自动故障迁移操作， 它会将失效主服务器的其中 一个从服务器升级为新的主服务器， 并让失效主服务器的其他从服务器改为复制新的主服务器; 当客 户端试图连接失效的主服务器时， 集群也会向客户端返回新主服务器的地址， 使得集群可以使用新主 服务器代替失效服务器。

### 配置

 **配置一：sentinel monitor <master-name> <ip> <port> <quorum>**

 这个配置表达的是 哨兵节点定期监控 名字叫做 <master-name>  并且 IP 为 <ip> 端口号为 <port> 的主节点。<quorum> 表示的是哨兵判断主节点是否发生故障的票数。也就是说如果我们将<quorum>设置为2就代表至少要有两个哨兵认为主节点故障了，才算这个主节点是客观下线的了，一般是设置为sentinel节点数的一半加一。

**配置二：sentinel down-after-milliseconds <master-name> <times>**

每个哨兵节点会定期发送ping命令来判断Redis节点和其余的哨兵节点是否是可达的，如果超过了配置的<times>时间没有收到pong回复，就主观判断节点是不可达的,<times>的单位为毫秒。

**配置三：sentinel parallel-syncs <master-name> <nums>**

当哨兵节点都认为主节点故障时，哨兵投票选出的leader会进行故障转移，选出新的主节点，原来的从节点们会向新的主节点发起复制，这个配置就是控制在故障转移之后，每次可以向新的主节点发起复制的节点的个数，最多为<nums>个，因为如果不加控制会对主节点的网络和磁盘IO资源很大的开销。

**配置四：sentinel failover-timeout <master-name>  <times>**

 这个代表哨兵进行故障转移时如果超过了配置的<times>时间就表示故障转移超时失败。

**配置五： sentinel auth-pass <master-name> <password>**

### 主观下线与与客观下线

sentinel对于不可用有两种不同的看法，一个叫主观不可用(SDOWN),另外一个叫客观不可用(ODOWN)。

主观不可用是sentinel自己主观上检测到的关于master的状态。

客观不可用需要一定数量的sentinel达成一致意见才能认为一个master客观上已经宕掉，各个sentinel之间通过命令 **SENTINEL is_master_down_by_addr** 来获得其它sentinel对master的检测结果。

从sentinel的角度来看，如果发送了PING心跳后，在一定时间内没有收到合法的回复，就达到了SDOWN的条件。这个时间在配置中通过 **is-master-down-after-milliseconds** 参数配置。

当sentinel发送PING后，以下回复都被认为是合法的,除此之外，其它任何回复（或者根本没有回复）都是不合法的。

### **Sentinel之间和Slaves之间的自动发现机制**

虽然sentinel集群中各个sentinel都互相连接彼此来检查对方的可用性以及互相发送消息。但是你不用在任何一个sentinel配置任何其它的sentinel的节点。因为sentinel利用了master的发布/订阅机制去自动发现其它也监控了统一master的sentinel节点。

通过向名为`__sentinel__:hello`的管道中发送消息来实现。

同样，你也不需要在sentinel中配置某个master的所有slave的地址，sentinel会通过询问master来得到这些slave的地址的。

每个sentinel通过向每个master和slave的发布/订阅频道`__sentinel__:hello`每秒发送一次消息，来宣布它的存在。
每个sentinel也订阅了每个master和slave的频道`__sentinel__:hello`的内容，来发现未知的sentinel，当检测到了新的sentinel，则将其加入到自身维护的master监控列表中。
每个sentinel发送的消息中也包含了其当前维护的最新的master配置。如果某个sentinel发现
自己的配置版本低于接收到的配置版本，则会用新的配置更新自己的master配置。

在为一个master添加一个新的sentinel前，sentinel总是检查是否已经有sentinel与新的sentinel的进程号或者是地址是一样的。如果是那样，这个sentinel将会被删除，而把新的sentinel添加上去。

- 

  第一行的格式如下：

  ```
  sentinel monitor [master-group-name] [ip] [port] [quorum]
  ```

  master-group-name：master名称

  quorun：本文叫做票数，Sentinel需要协商同意master是否可到达的数量。

  ```
  sentinel monitor mymaster 127.0.0.1 6379 2
  ```

  这一行用于告诉Redis监控一个master叫做mymaster，它的地址在127.0.0.1，端口为6379，票数是2。

  这里的票数需要解释下：举个栗子，redis集群中有5个sentinel实例，其中master挂掉啦，如果这里的票数是2，表示有2个sentinel认为master挂掉啦，才能被认为是正真的挂掉啦。其中sentinel集群中各个sentinel也有互相通信，通过gossip协议。

  除啦第一行其他的格式如下：

  ```
  sentinel [option_name] [master_name] [option_value]
  ```

  - **down-after-milliseconds**
    sentinel会向master发送心跳PING来确认master是否存活，如果master在“一定时间范围”内不回应PONG 或者是回复了一个错误消息，那么这个sentinel会主观地认为这个master已经不可用了。而这个down-after-milliseconds就是用来指定这个“一定时间范围”的，单位是毫秒。

  - **parallel-syncs**
    在发生failover主从切换时，这个选项指定了最多可以有多少个slave同时对新的master进行同步，这个数字越小，完成主从故障转移所需的时间就越长，但是如果这个数字越大，就意味着越多的slave因为主从同步而不可用。可以通过将这个值设为1来保证每次只有一个slave处于不能处理命令请求的状态。

   如果主节点设置了密码，则需要这个配置，否则哨兵无法对主节点进行监控。

## Cluster模式

### 正常启动redis服务

配置文件

```
port 设置端口 

daemonize  设置是否以守护进程开启

dir  设置目录，例如日志，rdb文件等等

dbfilename  设置rdb文件名

logfile   设置日志文件名

cluster-enabled yes  是否开启集群模式

cluster-config-file  集群节点的单独配置

cluster-node-timeout  超时时间，有很多用处，例如ping的有效时间，一般默认配置即可，如果ping超过，则认为节点不可用

cluster-require-full-coverage 用来设置什么情况下集群可以对外提供服务，yes表示只有集群节点

都正常才提供对外服务，一般生产环境设置为no

```

redis-server + 配置文件启动redis服务

### 配置集群互相发现

meet配置

```
redis-cli -h 127.0.0.1 -p 6379  cluster meet 127.0.0.1 6380

连接6379端口的服务器，按照上述方式依次对7001-7005（集群中其他节点）的端口进行握手
```

### hash槽配置

redis总共有16384个槽点，并且只有主节点需要分配槽点

```
 redis-cli -h 127.0.0.1 -p 6379 cluster addslots 0 1 2
```

### 主从配置 

```
redis-cli -h 127.0.0.1 -p 7003（从节点端口号） cluster replicate nodeid（主节点nodeid）
```

注意nodeid和runid的区别，nodeid重启之后也是不会改变的，但是runid一般重启之后就会改变。那么如果获取nodeid呢，执行如下命令：

```
redis-cli -p 6379 cluster nodes
```

### 集群扩容

步骤 

- 启动新节点
- meet操作加入急群众
- 为新节点分配hash槽（也就是数据迁移），把对应槽的数据迁移到新节点上（源节点槽及数据--->新节点），建议使用ruby reshard命令进行迁移

### 集群缩容

- 下线迁移槽

使用ruby的reshard命令

- 忘记节点

  先下从节点，在下主节点，不然会触发自动故障转移

   在需要忘记其他节点的客户端上执行命令如6379 忘记7000节点，在6379节点执行命令

  ```
   cluster forget(downNodeId) 被忘记节点ID
  ```

  ruby命令删除节点命令

- 关闭节点

### moved重定向和ASK重定向

**1、moved**

客户端发送命令给集群中的节点，经过crc计算，如果落入的槽在自己的节点内，则执行命令，反之给客户端发送真正的目标节点，然后在重新向真正的节点发送命令

**发生在槽迁移结束**

算出对应key的槽命令

```
127.0.0.1:7002> cluster keyslot hello
(integer) 866


127.0.0.1:7002> get hello
(error) MOVED 866 127.0.0.1:7000
```

**2、ASK**

如果发送命令是真正的节点，但是正在进行槽迁移，**发生在槽还在迁移中**

### 故障发现

通过ping/pong消息实现故障发现：不需要sentinel

#### 主观下线

某个节点任务另一个节点不可用

#### 主观下线

半数以上节点任务不可用

#### 故障恢复

让故障的主节点的从节点成为主节点

- 资格检查

每个从节点与故障主节点短线时间，超过cluster-node-timeout cluster-salve-validity-factory取消资格

- 准备选举时间
- 选举投票
- 让从节点替换主节点

### 集群限制

key批量操作支持有限：例如 mget mset必须在一个slot

key事务和Lua支持：操作的key必须在一个节点

大多数情况下不需要集群架构，哨兵即可







