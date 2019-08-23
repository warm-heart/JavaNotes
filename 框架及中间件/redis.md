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

##### 解决方案

- 缓存key，但数据为空。
- 对所有可能对应数据为空的key进行过滤（布隆过滤器），

### 缓存雪崩

由于缓存原因，导致大量请求发送到数据库。原因：缓存系统宕机。





