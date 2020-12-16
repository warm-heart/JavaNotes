# ElasticSearch 基础概念

## 基本概念

**索引**： 含有相同属性的文档集合，比如产品的索引，用户的索引等。索引在Es中是通过一个名字来识别的，而且它的名字必须是 英文字母小写，且不能有中划线。都是通过这个名字来进行增删查改。

**类型**：索引可以定义一个或多个类型，文档必须属于一个类型（一般会定义有相同字段的文档为一个类型）

**文档**：文档是可以被所有的基本数据单位，是整个ES中最小的存储单位

## 分片

ES默认在创建索引的时候，会创建5个分片，会为每个分片创建1个备份。这个默认的数量是可以修改的。

 number_of_shards： 每个索引的主分片数，默认值是 5 。这个配置在索引创建后不能修改。

 number_of_replicas：每个主分片的副本数，默认值是 1 。对于活动的索引库，这个配置可以随时修改。

# ElasticSearch 基本用法

## ES是以Restful API的风格来命名的。

- API 的基本格式

  

  ```
  http://:/<索引>/<类型>/<文档id>
  ```

  

- 常用HTTP动词

  ```
  GET/PUT/POST/DELETE
  ```

  

- 创建索引

  ```
  {
    "settings": {
    //刷新到osCache时间，也就是可以被搜索到
      "refresh_interval": "5",
      "index.routing.allocation.require.temperature": "hot",
      "number_of_shards": 1,
      "number_of_replicas": 0
    },
    "mappings": {
      "book": {
        "dynamic": false,
        "properties": {
          "bookId": {
            "type": "text"
          },
          "bookName": {
            "type": "text",
            "analyzer": "ik_max_word",
            "search_analyzer": "ik_max_word"
          },
          "bookDescription": {
            "type": "text",
            "analyzer": "ik_max_word",
            "search_analyzer": "ik_max_word"
          },
          "bookStock": {
            "type": "integer"
          },
          "bookPrice": {
            "type": "double"
          },
          "categoryName": {
            "type": "text",
            "analyzer": "ik_max_word",
            "search_analyzer": "ik_max_word"
          },
          "createTime": {
            "type": "date",
            "format": "strict_date_optional_time||epoch_millis"
          }
        }
      }
    }
  }
  ```

# 深分页

### form size

### scroll

该scroll参数（传递到search请求和每个scroll 请求）告诉Elasticsearch应该保持多长时间的搜索上下文活着。
 它的值（例如1m，参见“ 时间单位”）不需要足够长的时间来处理所有数据，而只需要足够长的时间来处理前一批结果即可。
 每个scroll请求（带有scroll参数）都设置一个新的到期时间。如果scroll没有在scroll 参数中传递请求，
 那么搜索上下文将作为该 scroll 请求的一部分被释放。scroll超过超时时间后，搜索上下文将自动删除。但是，保持滚动打开是有代价的（花费内存），
 因此，一旦不再使用clear-scrollAPI 使用滚动，则应明确清除滚动 

**缺点：scroll 会产生快照可能导致oom 并且不是实时的**

### search After

search After 为什么不用产生临时快照之类的存储信息，就能保证滚动顺序读取数据呢？
每个文档具有一个唯一值的字段应该用作排序规范的仲裁器。否则，具有相同排序值的文档的排序顺序将是未定义的。建议的方法是使用字段_id，它肯定包含每个文档的一个唯一值。

# 聚合

聚合相当于关系数据库中的group By

es不能对text字段进行聚合（原因是text字段不会生成DocValues）

# docValue

**docValue是正排索引结构，本质的目的是为了通过docid快速拿到所需要的字段值。**

1. 所有支持`doc_values`参数的字段都默认启用了。如果你确定不需要在一个字段上排序、聚合或通过脚本字段访问到该字段的值，你可以禁用`doc_values`，以节省磁盘空间，mapping设置："doc_values": false 禁用docValue，

2. DocValues其实是Lucene在构建索引时，会额外建立一个有序的基于document => field value的映射列表；

3. docValue是一种记录doc字段值的一种形式，在例如在结果排序和聚合查询时，需要**通过docid取字段值**的场景下是非常高效的。

4. 基于Lucene的es都是使用经典的倒排索引模式来达到快速检索的目的，简单的说就是建立 搜索词>>>>> 文档id列表 这样的关系映射，
   然后在搜索时，通过类似hash算法，来快速定位到一个搜索关键词，然后读取其的文档id集合，这就是倒排索引的核心思想，这样搜索数据
   是非常高效快速的，当然它也是有缺陷的，假如我们需要对数据做一些聚合操作，比**如排序，聚合时，如果没有正排索引时，Lucene内部会遍历倒排索引提取所有需要排序字段或者聚合字段然后再次构建一个最终的排好序的文档集合list**，这个步骤的过程全部维持在内存中操作，而且如果数据量巨大的话，非常容易就造成内存溢出和性能缓慢。

**如果有了正排索引，通过筛选过后拿到的docid去正排索引（DocValues）拿到所需字段的值进行聚合、排序性能是非常高的。**

# ES时间问题

## UTC协调世界

UTC协调世界时即格林威治平太阳时间，是指格林威治所在地的标准时间，也是表示地球自转速率的一种形式，UTC基于国际原子时间。

```
 //比如北京时间2020-01-01 08：00：00
 
 以0时区转结果为 utc时间 2020-01-01 08：00：00毫秒数
 public final static Long toEpochMilliFor0(LocalDateTime localDateTime) {
        Long second = localDateTime.toInstant(ZoneOffset.of("+0")).toEpochMilli();
        return second;
    }
    
     以东八区转结果为 utc时间 2020-01-01 00：00：00 毫秒数
    public final static Long toEpochMilli(LocalDateTime localDateTime) {
        Long second = localDateTime.toInstant(ZoneOffset.of("+8")).toEpochMilli();
        return second;
    }
```



## 解决方案

ES中最终会把时间转化为utc时间戳，也就是0时区时间（utc），我们再Java api查询的时候最好使用不带时区的时间（LocalDateTime）或者把时间转化为utc时间戳去查询（**不要转化为东八区时间戳**）

```
 //转化为0时区的时间戳
 Long second = localDateTime.toInstant(ZoneOffset.of("+0")).toEpochMilli();
```



# 写入原理

![5490117-da320c66996dd424](C:\Users\wql\Desktop\5490117-da320c66996dd424.png)

数据写入到内存buffer； 
同时写入到数据到translog buffer； 
每隔1s（refresh_interval）数据从buffer中refresh到FileSystemCache（OS cache）中，生成segment文件，一旦生成segment文件，就能通过索引查询到了； 
refresh完，memory buffer就清空了； 
每隔5s中，translog 从buffer flush到磁盘中； 
定期/定量从FileSystemCache中,结合translog内容 flush index到磁盘中。做增量flush的；

1. 数据先写入到buffer里面，在buffer里面的数据时搜索不到的，同时将数据写入到translog日志文件之中
2. 如果buffer快满了，或是一段时间之后，就会将buffer数据refresh到一个新的OS cache之中，然后每隔1秒，就会将OS cache的数据写入到segment file之中，但是如果每一秒钟没有新的数据到buffer之中，就会创建一个新的空的segment file，只要buffer中的数据被refresh到OS cache之中，就代表这个数据可以被搜索到了。当然可以通过restful api 和Java api，手动的执行一次refresh操作，就是手动的将buffer中的数据刷入到OS cache之中，让数据立马搜索到，只要数据被输入到OS cache之中，buffer的内容就会被清空了。同时进行的是，数据到shard之后，就会将数据写入到translog之中，每隔5秒将translog之中的数据持久化到磁盘之中
3. 重复以上的操作，每次一条数据写入buffer，同时会写入一条日志到translog日志文件之中去，这个translog文件会不断的变大，当达到一定的程度之后，就会触发commit操作。
4. 将一个commit point写入到磁盘文件，里面标识着这个commit point 对应的所有segment file
5. 强行将OS cache 之中的数据都fsync到磁盘文件中去。
   解释：translog的作用：在执行commit之前，所有的而数据都是停留在buffer或OS cache之中，无论buffer或OS cache都是内存，一旦这台机器死了，内存的数据就会丢失，所以需要将数据对应的操作写入一个专门的日志问价之中，一旦机器出现宕机，再次重启的时候，es会主动的读取translog之中的日志文件的数据，恢复到内存buffer和OS cache之中。
6. 将现有的translog文件进行清空，然后在重新启动一个translog，此时commit就算是成功了，默认的是每隔30分钟进行一次commit，但是如果translog的文件过大，也会触发commit，整个commit过程就叫做一个flush操作，我们也可以通过ES API,手动执行flush操作，手动将OS cache 的数据fsync到磁盘上面去，记录一个commit point，清空translog文件
   补充：其实translog的数据也是先写入到OS cache之中的，默认每隔5秒之中将数据刷新到硬盘中去，也就是说，可能有5秒的数据仅仅停留在buffer或者translog文件的OS cache中，如果此时机器挂了，会丢失5秒的数据，但是这样的性能比较好，我们也可以将每次的操作都必须是直接fsync到磁盘，但是性能会比较差。
7. 如果是删除操作，commit的时候会产生一个.del文件，里面讲某个doc标记为delete状态，那么搜索的时候，会根据.del文件的状态，就知道那个文件被删除了。
8. 如果时更新操作，就是讲原来的doc标识为delete状态，然后重新写入一条数据即可。
9. buffer每次更新一次，就会产生一个segment file 文件，所以在默认情况之下，就会产生很多的segment file 文件，将会定期执行merge操作
10. 每次merge的时候，就会将多个segment file 文件进行合并为一个，同时将标记为delete的文件进行删除，然后将新的segment file 文件写入到磁盘，这里会写一个commit point，标识所有的新的segment file，然后打开新的segment file供搜索使用。

# refresh_interval

当数据添加到索引后并不能马上被查询到，等到索引刷新后才会被查询到。 refresh_interval 也就是写入JVM buffer中，再刷新到osCache中的时间间隔，

## `-1` 的使用

当 refresh_interval 为 -1 时，意味着不刷新索引。当需要大量导入数据到ES中，可以将 refresh_interval 设置为 -1 以加快导入速度。导入结束后，再将 refresh_interval 设置为一个正数，例如`1s`。或者手动 refresh 索引。

假如refresh_interval设置为-1，是不是永远不会刷新导数据一直在内存里，内存不就炸了？

1. 在一个index请求，处理完成之后会根据缓存区大小判断是否需要进行refresh操作。如果满了就会进行refresh操作，所以不会操作内存溢出

# 读取原理

查询过程大体上分为查询和取回这两个阶段，广播查询请求到所有相关分片，并将它们的响应整合成全局排序后的结果集合，这个结果集合会返回给客户端。

1. 查询阶段
   1. 当一个节点接收到一个搜索请求，这这个节点就会变成协调节点，第一步就是将广播请求到搜索的每一个节点的分片拷贝，查询请求可以被某一个主分片或某一个副分片处理，协调节点将在之后的请求中轮训所有的分片拷贝来分摊负载。
   2. 每一个分片将会在本地构建一个优先级队列，如果客户端要求返回结果排序中从from 名开始的数量为size的结果集，每一个节点都会产生一个from+size大小的结果集，因此优先级队列的大小也就是from+size，分片仅仅是返回一个轻量级的结果给协调节点，包括结果级中的每一个文档的ID和进行排序所需要的信息。
   3. 协调节点将会将所有的结果进行汇总，并进行全局排序，最总得到排序结果。
2. 取值阶段
   1. 查询过程得到的排序结果，标记处哪些文档是符合要求的，此时仍然需要获取这些文档返回给客户端
   2. 协调节点会确定实际需要的返回的文档，并向含有该文档的分片发送get请求，分片获取的文档返回给协调节点，协调节点将结果返回给客户端。

# ES优化

## 断路器

```
# 缓存回收大小，无默认值
# 有了这个设置，最久未使用（LRU）的 fielddata 会被回收为新数据腾出空间
# 控制fielddata允许内存大小，达到HEAP 20% 自动清理旧cache 
# 如果这个参数小于fielddata断路器参数，则永远不会触发fielddata断路器
indices.fielddata.cache.size: 20%


indices.breaker.total.use_real_memory: false
# 总断路器 揉合 request 和 fielddata 断路器保证两者组合起来不会使用超过堆内存的 70%(默认值)。
#如果indices.breaker.total.use_real_memory为 false，则默认为JVM堆的70％ false。如果indices.breaker.total.use_real_memory 为true，则默认为JVM堆的95％。
indices.breaker.total.limit: 85%


# fielddata断路器估算将fielddata数据加载到fielddata高速缓存所需的堆内存。如果加载该字段将导致缓存超过预定义的内存限制，则断路器将停止操作并返回错误。

#fielddata数据断路器的限制。默认为JVM堆的40％。
indices.breaker.fielddata.limit: 40%
#与所有字段数据估计值相乘以确定最终估计值的常数。默认为1.03。
indices.breaker.fielddata.overhead: 1.03


# request 断路器估算需要完成其他请求部分的结构大小，例如，用于在请求期间计算聚合的内存，默认为JVM堆的60％。
indices.breaker.request.limit: 40%
 #一个常数，所有请求估计值都将与该常数相乘以确定最终估计值。默认为1
indices.breaker.request.overhead: 1

#请求缓存设置 设置为JVM堆的2%，默认1%
indices.requests.cache.size: 2%
```

## JVM

堆内存不小于2G，不大于32G（大于32G会关闭指针压缩。 每个对象的指针都变长了，就会使用更多的 CPU 内存带宽，也就是说你实际上失去了更多的内存。）

- 设置`Xmx`和`Xms`不超过物理内存50％。Elasticsearch除了JVM堆以外的目的而需要内存，因此为此留出空间很重要。例如，Elasticsearch使用堆外缓冲区来进行有效的网络通信，依靠操作系统的文件系统缓存来有效地访问文件，并且JVM本身也需要一些内存。观察Elasticsearch过程使用的内存多于该`Xmx`设置配置的限制，这是正常的。

## 分片请求缓存

当针对一个索引或多个索引运行搜索请求时，每个涉及的分片都将在本地执行搜索，并将其本地结果返回给*协调节点*，该节点将这些分片级结果合并为一个全局结果集。

分片级请求缓存模块在每个分片上缓存本地结果。这允许频繁使用（并且可能复杂）的搜索请求几乎立即返回结果。请求高速缓存非常适合日志记录用例，在这种情况下，只有最新索引才被主动更新-旧索引的结果将直接从高速缓存中提供。

### 缓存设置

缓存是在节点级别管理的，并且具有`1%` 堆的默认最大大小。可以在`config/elasticsearch.yml`文件中进行更改，此外，也可以使用该`indices.requests.cache.expire`设置为缓存的结果指定TTL，但是没有理由这样做。因为刷新索引后，旧的结果将自动失效。

### 缓存失效机制

缓存是智能的-保证未缓存搜索与缓存搜索有着几乎相同的结果；

每当**分片刷新**时，高速缓存的结果都会自动失效，但**前提是分片中的数据实际上已更改**。换句话说，始终会从缓存中获得与未缓存的搜索请求相同的结果。

缓存可以使用restfulAPI设置失效

```
POST  localhost:9200/my-index-000001/_cache/clear?request=true
```

### 开启和禁用缓存

缓存默认情况下处于启用状态，但是在创建新索引时可以将其禁用，如下所示：

```
PUT /my-index-000001
{
  "settings": {
    "index.requests.cache.enable": false
  }
}
```

也可以使用修改索引设置API在现有索引上动态启用或禁用它 ：

```
PUT /my-index-000001/_settings
{ "index.requests.cache.enable": true }
```

### 在每个请求上启用和禁用缓存

```
GET /my-index-000001/_search?request_cache=true
{
  "size": 0,
  "aggs": {
    "popular_colors": {
      "terms": {
        "field": "colors"
      }
    }
  }
}
```

### 查看缓存使用大小

缓存的大小（以字节为单位）和逐出的数量可以使用api查看

```
GET /_stats/request_cache?human
```

# forceMerge

使用强制合并API可以在一个或多个索引的分片上强制进行合并，合并通过将每个分片中的某些片段合并在一起来减少其数量，还可以释放已删除文档所占用的空间。合并通常自动发生，但有时手动触发合并很有用。

大量删除操作后，es会记录删除数据，反而会增加磁盘空间，这时进行forceMerge操作，会物理删除已经删除的数据。

**注意：**强制合并会导致产生非常大的段（segments ）（> 5GB），并且，如果您继续写入这样的索引，则自动合并策略将永远不会考虑这些段用于将来的合并，直到它们主要由已删除的文档组成。这会导致很大的段保留在索引中，从而导致磁盘使用率增加和搜索性能下降。所以慎用。

##  强制合并期间发生阻塞

调用forceMerge，直到合并完成。如果客户端连接在完成前丢失，则强制合并过程将在后台继续。强制合并相同索引的任何新请求也将阻止，直到正在进行的强制合并完成为止。也就是强制合并期间不能进行搜索。

# ES冷热分离



# Spring Boot整合ElasticSearch

#### 加入依赖

```
<dependency>
            <groupId>org.elasticsearch</groupId>
            <artifactId>elasticsearch</artifactId>
            <version>${elasticSearch.version}</version>
        </dependency>
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-high-level-client</artifactId>
            <version>${elasticSearch.version}</version>
        </dependency>
        <dependency>
            <groupId>org.elasticsearch.plugin</groupId>
            <artifactId>transport-netty4-client</artifactId>
            <version>${elasticSearch.version}</version>
        </dependency>
```

#### 配置RestHighLevelClient

```
@Configuration
public class ESConfig {

    private static String hosts = "127.0.0.1"; // 集群地址，多个用,隔开
    private static int port = 9200; // 使用的端口号
    private static String schema = "http"; // 使用的协议
    private static ArrayList<HttpHost> hostList = null;
    private static int connectTimeOut = 1000; // 连接超时时间
    private static int socketTimeOut = 30000; // 连接超时时间
    private static int connectionRequestTimeOut = 500; // 获取连接的超时时间
    private static int maxConnectNum = 100; // 最大连接数
    private static int maxConnectPerRoute = 100; // 最大路由连接数

    static {
        hostList = new ArrayList<>();
        String[] hostStrs = hosts.split(",");
        for (String host : hostStrs) {
            hostList.add(new HttpHost(host, port, schema));
        }
    }

    @Bean
    public RestHighLevelClient client() {
        RestClientBuilder builder = RestClient.builder(hostList.toArray(new HttpHost[0]));
        // 异步httpclient连接延时配置
        builder.setRequestConfigCallback(new RequestConfigCallback() {
            @Override
            public Builder customizeRequestConfig(Builder requestConfigBuilder) {
                requestConfigBuilder.setConnectTimeout(connectTimeOut);
                requestConfigBuilder.setSocketTimeout(socketTimeOut);
                requestConfigBuilder.setConnectionRequestTimeout(connectionRequestTimeOut);
                return requestConfigBuilder;
            }
        });

        // 异步httpclient连接数配置
        builder.setHttpClientConfigCallback(new HttpClientConfigCallback() {
            @Override
            public HttpAsyncClientBuilder customizeHttpClient(HttpAsyncClientBuilder httpClientBuilder) {
                httpClientBuilder.setMaxConnTotal(maxConnectNum);
                httpClientBuilder.setMaxConnPerRoute(maxConnectPerRoute);
                return httpClientBuilder;
            }
        });
        RestHighLevelClient client = new RestHighLevelClient(builder);
        return client;

    }
}
```

spring Boot整合ElasticSearch操作数据

#### 数据的插入

```
BookIndexTemplate bookIndexTemplate = new BookIndexTemplate();
bookIndexTemplate.setBookId("2");
bookIndexTemplate.setBookName("西游记");
bookIndexTemplate.setBookDescription("中国最古典的小说");
bookIndexTemplate.setBookStock(100);
bookIndexTemplate.setBookPrice(59.9);
bookIndexTemplate.setCategoryName("小说");
bookIndexTemplate.setCreateTime(new Date());

IndexRequest indexRequest = new IndexRequest(EsConsts.INDEX_NAME, EsConsts.TYPE, bookIndexTemplate.getBookId());
indexRequest.source(JSON.toJSONString(bookIndexTemplate), XContentType.JSON);
IndexResponse indexResponse = client.index(indexRequest, RequestOptions.DEFAULT);


System.out.println("add: " + JSON.toJSONString(indexResponse));
```

####  数据的修改

```
BookIndexTemplate bookIndexTemplate = new BookIndexTemplate();
bookIndexTemplate.setBookId("4");
bookIndexTemplate.setBookName("红楼梦");
bookIndexTemplate.setBookDescription("中国最古典的小说");
bookIndexTemplate.setBookStock(100);
bookIndexTemplate.setBookPrice(66.9);
bookIndexTemplate.setCategoryName("小说");
bookIndexTemplate.setCreateTime(new Date());

UpdateRequest request = new UpdateRequest(EsConsts.INDEX_NAME, EsConsts.TYPE, bookIndexTemplate.getBookId());
request.doc(JSON.toJSONString(bookIndexTemplate), XContentType.JSON);
UpdateResponse updateResponse = client.update(request, RequestOptions.DEFAULT);
System.out.println("update: " + JSON.toJSONString(updateResponse));
```

#### 数据的删除

```

        DeleteRequest deleteRequest =
                new DeleteRequest(EsConsts.INDEX_NAME, EsConsts.TYPE, new String("1").toString());
        DeleteResponse response = client.delete(deleteRequest, RequestOptions.DEFAULT);
        System.out.println("delete: " + JSON.toJSONString(response));
```

#### 数据的搜索

 ##### 高级查询

**termQuery和matchQuery区别：**

term不会分词，match会根据提供的分词器进行分词

**AND与OR：**

filter，must，should

- AND查询

可以多个条件一起相当于数据库中的 AND，两种实现filter和must

```
        BoolQueryBuilder boolBuilder = QueryBuilders.boolQuery();

        //可以多个条件一起相当于数据库中的 AND
        //boolBuilder.filter(QueryBuilders.termQuery(EsConsts.BOOK_DESCRIPTION, "古典"));
        //boolBuilder.filter(QueryBuilders.matchQuery(EsConsts.CATEGORY_NAME, "古典"));
        //和上面效果一样都相当于AND must的性能要低一些，因为他要进行打分评估，就是说要进行_score，而filter则不会
        //boolBuilder.must(QueryBuilders.termQuery(EsConsts.BOOK_DESCRIPTION, "小说"));
        //boolBuilder.must(QueryBuilders.matchQuery(EsConsts.CATEGORY_NAME, "小说"));
```

- 查询在时间区范围内结果

  ```
  RangeQueryBuilder rangbuilder = QueryBuilders.rangeQuery("createTime");
  SimpleDateFormat sDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"); //加上时间
  Date date = sDateFormat.parse("2019-08-07 16:30:00");
  Date date1 = sDateFormat.parse("2019-09-13 12:00:00");
  rangbuilder.gte(date);
  rangbuilder.lte(date1);
  //加入boolBuilder
  boolBuilder.must(rangbuilder);
  ```

- OR查询

可以多个条件一起相当于数据库中的 OR

```

        boolBuilder.should(QueryBuilders.termQuery(EsConsts.BOOK_DESCRIPTION, "小说"));
        boolBuilder.should(QueryBuilders.matchQuery(EsConsts.CATEGORY_NAME, "dafdfer"));
```

##### 排序

```

        FieldSortBuilder fsb = SortBuilders.fieldSort("date");
        fsb.order(SortOrder.DESC);
        //加入SearchSourceBuilder
        sourceBuilder.sort(fsb);
```

##### 查询索引映射成实体类完整实例

```
       // 构建查询条件
        BoolQueryBuilder boolBuilder = QueryBuilders.boolQuery();

        //可以多个条件一起相当于数据库中的 AND
        //boolBuilder.filter(QueryBuilders.termQuery(EsConsts.BOOK_DESCRIPTION, "古典"));
        //boolBuilder.filter(QueryBuilders.matchQuery(EsConsts.CATEGORY_NAME, "古典"));
        //和上面效果一样都相当于AND must的性能要低一些，因为他要进行打分评估，就是说要进行_score，而filter则不会
        //boolBuilder.must(QueryBuilders.termQuery(EsConsts.BOOK_DESCRIPTION, "小说"));
        //boolBuilder.must(QueryBuilders.matchQuery(EsConsts.CATEGORY_NAME, "小说"));

        //可以多个条件一起相当于数据库中的 OR
        boolBuilder.should(QueryBuilders.termQuery(EsConsts.BOOK_DESCRIPTION, "小说"));
        boolBuilder.should(QueryBuilders.matchQuery(EsConsts.CATEGORY_NAME, "dafdfer"));
        
        
        
        //构建SearchSourceBuilder
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        sourceBuilder.query(boolBuilder);
        sourceBuilder.highlighter(highlightBuilder);

        sourceBuilder.from(0);
        // 获取记录数，默认10
        sourceBuilder.size(100);

        // 第一个是获取字段，第二个是过滤的字段，默认获取全部
        //sourceBuilder.fetchSource(new String[]{"bookId", "bookDescription","bookName"}, new String[]{});


        //构建SearchRequest
        SearchRequest searchRequest = new SearchRequest(EsConsts.INDEX_NAME);
        searchRequest.types(EsConsts.TYPE);
        searchRequest.source(sourceBuilder);
        //得到SearchResponse
        SearchResponse response = null;
        response = client.search(searchRequest, RequestOptions.DEFAULT);
        SearchHits hits = response.getHits();
        SearchHit[] searchHits = hits.getHits();
        
        //遍历数据并转换为实体类
         List<String> bookIndexTemplates = new ArrayList<>(100);
        for (SearchHit hit : searchHits) {
            System.out.println("search -> " + hit.getSourceAsString());
            BookIndexTemplate bookIndexTemplate =
                    JSONObject.parseObject(hit.getSourceAsString(), BookIndexTemplate.class);
            bookIndexTemplates.add(bookIndexTemplate.getBookId());
        }
        bookIndexTemplates.forEach(e -> System.out.println(e));

    }
```



​	
