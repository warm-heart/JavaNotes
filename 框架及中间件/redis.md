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

Redis增量复制是指Slave初始化后开始正常工作时主服务器发生的写操作同步到从服务器的过程。 
增量复制的过程主要是主服务器每执行一个写命令就会向从服务器发送相同的写命令，从服务器接收并执行收到的写命令。

**缺点：**

没有实现故障自动转移

## 哨兵模式



## 集群模式















