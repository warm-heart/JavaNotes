## Dubbo

*Dubbo* 是阿里巴巴为Java 开发的RPC 框架，据网上评价来看非常不错。在Dubbo 开源后很多公司都使用其来构建自己的分布式架构。

**Remoting:远程通讯**，提供对多种NIO框架抽象封装，包括“同步转异步”和“请求-响应”模式的信息交换方式。

**Cluster: 服务框架**，提供基于接口方法的透明远程过程调用，包括多协议支持，以及软负载均衡，失败容错，地址路由，动态配置等集群支持。

**Registry: 服务注册中心**，基于注册中心目录服务，使服务消费方能动态的查找服务提供方，使地址透明，使服务提供方可以平滑增加或减少机器。

### 组件角色

- Provider：暴露服务的服务提供方
- Consumer：调用远程服务的服务消费方
- Registry：服务注册与发现的注册中心（Zookeeper）
- Monitor：统计服务的调用次调和调用时间的监控中心
- Container：服务运行容器

### 调用关系

**服务容器Container**负责启动，加载，运行服务提供者。

**服务提供者Provider**在启动时，向注册中心注册自己提供的服务。

**服务消费者Consumer**在启动时，向注册中心订阅自己所需的服务。

**注册中心Registry**返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。

**服务消费者Consumer**，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。

**服务消费者Consumer和提供者Provider**，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心Monitor。

### 与spring结合

#### 与Spring Boot结合建立多模块项目



建立四大模块

- API：公共接口模块
- server：服务提供者一模块
- provide2：服务提供者二模块
- client：消费者模块

父模块依赖

```
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.3.RELEASE</version>
    </parent>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <!--dubbo依赖版本统一控制-->
       <spring-boot-dubbo-version>1.0.0</spring-boot-dubbo-version>
    </properties>
    <!--dependencyManagement标签 多maven项目依赖控制,
    dependencyManagement标签内的依赖，子pom不会直接继承，在子pom里需手动引入
    dependencyManagement标签外的依赖子pom会自动依赖-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>io.dubbo.springboot</groupId>
                <artifactId>spring-boot-starter-dubbo</artifactId>
                <version>${spring-boot-dubbo-version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <dependencies>
        <!--junit-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <!--springboot com.study.web-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--springboot test-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>

        <!--mysql驱动-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.46</version>
        </dependency>
        <!--druid 连接池-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.6</version>
        </dependency>
        <!--mybatis-->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.3.1</version>
        </dependency>
    </dependencies>
```

#### API模块

不需要任何依赖，只定义接口，作为jar包宫其他模块引用

#### 服务消费者

服务提供者与服务消费者都需要dubbo依赖和API模块的依赖和zookeeper依赖如下：

```
 <dependency>
            <groupId>com.study</groupId>
            <artifactId>API</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <!--dubbo-springBoot依赖-->
        <dependency>
            <groupId>io.dubbo.springboot</groupId>
            <artifactId>spring-boot-starter-dubbo</artifactId>
        </dependency>
        <!--zookeeper依赖-->
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.8</version>
        </dependency>
```

配置文件如下：

```
spring.dubbo.application.name=consumer 
spring.dubbo.registry.address=zookeeper://127.0.0.1:2181 
spring.dubbo.scan=com.study.Service
server.port=8889
```

#### 服务提供者

需要依赖和服务消费者一样

配置文件

```
//服务名
spring.dubbo.application.name=provider
//注册中心地址
spring.dubbo.registry.address=zookeeper://127.0.0.1:2181
//协议名称
spring.dubbo.protocol.name=dubbo
//注册端口，多个服务不能再同一个端口注册
spring.dubbo.protocol.port=20880
//扫描实现了API接口的类
spring.dubbo.scan=com.study.provide
//spring端口 和dubbo无关
server.port=8888
```

#### 启动步骤：

一 启动zookeeper
二 启动 Springboot Provide
三 启动 Springboot Consume
四 访问 Provide 和Consume的web路由

### 一个接口多个实现

server模块与provide2模块是对同一接口不同实现

##### 分组

对于dubbo 一个接口有多个实现可以使用group进行分组

- 两个服务提供者均未配置分组，消费者会按照负载均衡策略进行轮询调用

- 如果提供者指定group ，那么消费者必须指定group，提供者端可以设置权重，注解中的weight属性

  服务提供方2

```
@Service(group = "provide2")
```

   服务提供方1

```
@Service(group = "provide1")
```

消费者

- 调用提供者2的实现

  ```
  @Reference(group = "provide2")
  ```

- 调用提供者1的实现

```
  @Reference(group = "provide2")
```



