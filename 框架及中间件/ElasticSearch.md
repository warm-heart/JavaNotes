## 一 、ElasticSearch 基础概念

### 基本概念

**索引**： 含有相同属性的文档集合，比如产品的索引，用户的索引等。索引在Es中是通过一个名字来识别的，而且它的名字必须是 英文字母小写，且不能有中划线。都是通过这个名字来进行增删查改。

**类型**：索引可以定义一个或多个类型，文档必须属于一个类型（一般会定义有相同字段的文档为一个类型）

**文档**：文档是可以被所有的基本数据单位，是整个ES中最小的存储单位

### 分片

ES默认在创建索引的时候，会创建5个分片，会为每个分片创建1个备份。这个默认的数量是可以修改的。

 number_of_shards： 每个索引的主分片数，默认值是 5 。这个配置在索引创建后不能修改。

 number_of_replicas：每个主分片的副本数，默认值是 1 。对于活动的索引库，这个配置可以随时修改。

## 二、 ElasticSearch 基本用法

#### ES是以Restful API的风格来命名的。

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

## 三 、Spring Boot整合ElasticSearch

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

#### 排序

```

        FieldSortBuilder fsb = SortBuilders.fieldSort("date");
        fsb.order(SortOrder.DESC);
        //加入SearchSourceBuilder
        sourceBuilder.sort(fsb);
```



### 查询索引映射成实体类完整实例

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