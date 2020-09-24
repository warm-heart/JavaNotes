# eureka

## 自我保护机制

Eureka Server以一种谦虚的态度来判断是不是自己出了问题，也就是当大量的服务续约超时时，就认为是自己出现问题了。（如果少量服务续约超时，则认为是服务故障），在实际中有可能出现网络闪断情况，Eureka Server通过判断是否有大量续约失败，来确定是否开启自我保护，判断上一分钟的续约数是否小于自我保护阀值，如果上一分钟的续约数小于自我保护阀值，则开启自我保护机制，不再进行服务的剔除，一旦开启了保护机制，则服务注册中心维护的服务实例就不是那么准确了，此时可以使用`eureka.server.enable-self-preservation=false`来关闭保护机制，这样可以确保注册中心中不可用的实例被及时的剔除（**不推荐**）。

## 服务提供者

### 服务注册

服务提供者在启动的时候通过发送REST请求的方式将自己注册到Eureka Server上，同时带上自身元数据信息。 Eureka Server接收到这个REST请求之后，将元数据的信息储存在一个双层结构的Map中，第一层map的key为服务名，第二层的key为具体服务的实例名。

### 服务续约（Renew）

注册完服务之后，服务提供者会维持一个心跳告诉Eureka Server ：“我是一个活着的健康实例”，从而防止Eureka Server的“剔除任务”从服务列表中排除该实例。 eureka.instance.lease-renewal-interval-in-seconds：心跳任务的调用时间，默认三十秒， eureka.instance.lease-expiration-duration-seconds：服务时效时间，默认九十秒，超过时间剔除服务。

## 服务消费者

### 获取服务

服务消费者启动的时候会发送一个REST请求给注册中心，获取已注册的服务清单，并把服务清单放在本地缓存，eureka client每隔三十秒向eureka server拉取一次服务信息。 eureka.client.fetch-registry：获取服务，默认为true; eureka.client.registry-fetch-interval-seconds：缓存清单的更新时间，默认三十秒，网关可以设置短一点。

### 调用服务

Eureka先从本地缓存找调取福的信息，如果找不到就去eureka Server去拉取，就可以获得服务者的实例名和元数据信息。 Ribbon默认使用轮询的方式调用，从而实现客户端负载均衡

## 配置

Eureka包含四个部分的配置
instance：当前Eureka Instance实例信息配置
client：Eureka Client客户端特性配置
server：Eureka Server注册中心特性配置
dashboard：Eureka Server注册中心仪表盘配置

其中eureka.instance可以存在于eureka server和eureka client中，也就是说他是公共eureka实例的公共信息。

### instance配置

```

#服务注册中心实例的主机名
eureka.instance.hostname=localhost

#注册在Eureka服务中的应用组名
eureka.instance.app-group-name=

#注册在的Eureka服务中的应用名称
eureka.instance.appname=

#该实例注册到服务中心的唯一ID
eureka.instance.instance-id=

#该实例的IP地址
eureka.instance.ip-address=

#该实例，相较于hostname是否优先使用IP
eureka.instance.prefer-ip-address=false

#用于AWS平台自动扩展的与此实例关联的组名，
eureka.instance.a-s-g-name=

#部署此实例的数据中心
eureka.instance.data-center-info=

#默认的地址解析顺序
eureka.instance.default-address-resolution-order=

#该实例的环境配置
eureka.instance.environment=

#初始化该实例，注册到服务中心的初始状态
eureka.instance.initial-status=up

#表明是否只要此实例注册到服务中心，立马就进行通信
eureka.instance.instance-enabled-onit=false

#该服务实例的命名空间,用于查找属性
eureka.instance.namespace=eureka

#该服务实例的子定义元数据，可以被服务中心接受到
eureka.instance.metadata-map.test = test

 
#服务中心删除此服务实例的等待时间(秒为单位),时间间隔为最后一次服务中心接受到的心跳时间
eureka.instance.lease-expiration-duration-in-seconds=90

#该实例给服务中心发送心跳的间隔时间，用于表明该服务实例可用
eureka.instance.lease-renewal-interval-in-seconds=30

#该实例，注册服务中心，默认打开的通信数量
eureka.instance.registry.default-open-for-traffic-count=1

#每分钟续约次数
eureka.instance.registry.expected-number-of-renews-per-min=1

 

#该实例健康检查url,绝对路径
eureka.instance.health-check-url=

#该实例健康检查url,相对路径
eureka.instance.health-check-url-path=/health

#该实例的主页url,绝对路径
eureka.instance.home-page-url=

#该实例的主页url,相对路径
eureka.instance.home-page-url-path=/

#该实例的安全健康检查url,绝对路径
eureka.instance.secure-health-check-url=

#https通信端口
eureka.instance.secure-port=443

#https通信端口是否启用
eureka.instance.secure-port-enabled=false

#http通信端口
eureka.instance.non-secure-port=80

#http通信端口是否启用
eureka.instance.non-secure-port-enabled=true

#该实例的安全虚拟主机名称(https)
eureka.instance.secure-virtual-host-name=unknown

#该实例的虚拟主机名称(http)
eureka.instance.virtual-host-name=unknown

#该实例的状态呈现url,绝对路径
eureka.instance.status-page-url=

#该实例的状态呈现url,相对路径
eureka.instance.status-page-url-path=/status

```

### client配置

```

#该客户端是否可用
eureka.client.enabled=true

#实例是否在eureka服务器上注册自己的信息以供其他服务发现，默认为true
eureka.client.register-with-eureka=false

#此客户端是否获取eureka服务器注册表上的注册信息，默认为true
eureka.client.fetch-registry=false

#是否过滤掉，非UP的实例。默认为true
eureka.client.filter-only-up-instances=true

#与Eureka注册服务中心的通信zone和url地址
eureka.client.serviceUrl.defaultZone=http://${eureka.instance.hostname}:${server.port}/eureka/

#client连接Eureka服务端后的空闲等待时间，默认为30 秒
eureka.client.eureka-connection-idle-timeout-seconds=30

#client连接eureka服务端的连接超时时间，默认为5秒
eureka.client.eureka-server-connect-timeout-seconds=5

#client对服务端的读超时时长
eureka.client.eureka-server-read-timeout-seconds=8

#client连接all eureka服务端的总连接数，默认200
eureka.client.eureka-server-total-connections=200

#client连接eureka服务端的单机连接数量，默认50
eureka.client.eureka-server-total-connections-per-host=50

#执行程序指数回退刷新的相关属性，是重试延迟的最大倍数值，默认为10
eureka.client.cache-refresh-executor-exponential-back-off-bound=10

#执行程序缓存刷新线程池的大小，默认为5
eureka.client.cache-refresh-executor-thread-pool-size=2

#心跳执行程序回退相关的属性，是重试延迟的最大倍数值，默认为10
eureka.client.heartbeat-executor-exponential-back-off-bound=10

#心跳执行程序线程池的大小,默认为5
eureka.client.heartbeat-executor-thread-pool-size=5

# 询问Eureka服务url信息变化的频率（s），默认为300秒
eureka.client.eureka-service-url-poll-interval-seconds=300

#最初复制实例信息到eureka服务器所需的时间（s），默认为40秒
eureka.client.initial-instance-info-replication-interval-seconds=40

#间隔多长时间再次复制实例信息到eureka服务器，默认为30秒
eureka.client.instance-info-replication-interval-seconds=30

#从eureka服务器注册表中获取注册信息的时间间隔（s），默认为30秒
eureka.client.registry-fetch-interval-seconds=30
 

# 获取实例所在的地区。默认为us-east-1
eureka.client.region=us-east-1

#实例是否使用同一zone里的eureka服务器，默认为true，理想状态下，eureka客户端与服务端是在同一zone下
eureka.client.prefer-same-zone-eureka=true

# 获取实例所在的地区下可用性的区域列表，用逗号隔开。（AWS）
eureka.client.availability-zones.china=defaultZone,defaultZone1,defaultZone2

#eureka服务注册表信息里的以逗号隔开的地区名单，如果不这样返回这些地区名单，则客户端启动将会出错。默认为null
eureka.client.fetch-remote-regions-registry=

#服务器是否能够重定向客户端请求到备份服务器。 如果设置为false，服务器将直接处理请求，如果设置为true，它可能发送HTTP重定向到客户端。默认为false
eureka.client.allow-redirects=false

#客户端数据接收
eureka.client.client-data-accept=

#增量信息是否可以提供给客户端看，默认为false
eureka.client.disable-delta=false

#eureka服务器序列化/反序列化的信息中获取“_”符号的的替换字符串。默认为“__“
eureka.client.escape-char-replacement=__

#eureka服务器序列化/反序列化的信息中获取“$”符号的替换字符串。默认为“_-”
eureka.client.dollar-replacement="_-"

#当服务端支持压缩的情况下，是否支持从服务端获取的信息进行压缩。默认为true
eureka.client.g-zip-content=true

#是否记录eureka服务器和客户端之间在注册表的信息方面的差异，默认为false
eureka.client.log-delta-diff=false

# 如果设置为true,客户端的状态更新将会点播更新到远程服务器上，默认为true
eureka.client.on-demand-update-status-change=true

#此客户端只对一个单一的VIP注册表的信息感兴趣。默认为null
eureka.client.registry-refresh-single-vip-address=

#client是否在初始化阶段强行注册到服务中心，默认为false
eureka.client.should-enforce-registration-at-init=false

#client在shutdown的时候是否显示的注销服务从服务中心，默认为true
eureka.client.should-unregister-on-shutdown=true

# 获取eureka服务的代理主机，默认为null
eureka.client.proxy-host=

#获取eureka服务的代理密码，默认为null
eureka.client.proxy-password=

# 获取eureka服务的代理端口, 默认为null
eureka.client.proxy-port=

# 获取eureka服务的代理用户名，默认为null
eureka.client.proxy-user-name=

#属性解释器
eureka.client.property-resolver=

#获取实现了eureka客户端在第一次启动时读取注册表的信息作为回退选项的实现名称
eureka.client.backup-registry-impl=

#这是一个短暂的×××的配置，如果最新的×××是稳定的，则可以去除，默认为null
eureka.client.decoder-name=

#这是一个短暂的编码器的配置，如果最新的编码器是稳定的，则可以去除，默认为null
eureka.client.encoder-name=


#是否使用DNS机制去获取服务列表，然后进行通信。默认为false
eureka.client.use-dns-for-fetching-service-urls=false

#获取要查询的DNS名称来获得eureka服务器，此配置只有在eureka服务器ip地址列表是在DNS中才会用到。默认为null
eureka.client.eureka-server-d-n-s-name=

#获取eureka服务器的端口，此配置只有在eureka服务器ip地址列表是在DNS中才会用到。默认为null
eureka.client.eureka-server-port=

#表示eureka注册中心的路径，如果配置为eureka，则为http://x.x.x.x:x/eureka/，在eureka的配置文件中加入此配置表示eureka作为客户端向注册中心注册，从而构成eureka集群。此配置只有在eureka服务器ip地址列表是在DNS中才会用到，默认为null
eureka.client.eureka-server-u-r-l-context=

```

### server配置

```

#服务端开启自我保护模式。无论什么情况，服务端都会保持一定数量的服务。避免client与server的网络问题，而出现大量的服务被清除。
eureka.server.enable-self-preservation=true

#开启清除无效服务的定时任务，时间间隔。默认1分钟
eureka.server.eviction-interval-timer-in-ms= 60000

#间隔多长时间，清除过期的delta数据
eureka.server.delta-retention-timer-interval-in-ms=0

#过期数据，是否也提供给client
eureka.server.disable-delta=false

#eureka服务端是否记录client的身份header
eureka.server.log-identity-headers=true

#请求频率限制器
eureka.server.rate-limiter-burst-size=10

#是否开启请求频率限制器
eureka.server.rate-limiter-enabled=false

#请求频率的平均值
eureka.server.rate-limiter-full-fetch-average-rate=100

#是否对标准的client进行频率请求限制。如果是false，则只对非标准client进行限制
eureka.server.rate-limiter-throttle-standard-clients=false

#注册服务、拉去服务列表数据的请求频率的平均值
eureka.server.rate-limiter-registry-fetch-average-rate=500

#设置信任的client list
eureka.server.rate-limiter-privileged-clients=

#在设置的时间范围类，期望与client续约的百分比。
eureka.server.renewal-percent-threshold=0.85

#多长时间更新续约的阈值
eureka.server.renewal-threshold-update-interval-ms=0

#对于缓存的注册数据，多长时间过期
eureka.server.response-cache-auto-expiration-in-seconds=180

#多长时间更新一次缓存中的服务注册数据
eureka.server.response-cache-update-interval-ms=0

#缓存增量数据的时间，以便在检索的时候不丢失信息
eureka.server.retention-time-in-m-s-in-delta-queue=0

#当时间戳不一致的时候，是否进行同步
eureka.server.sync-when-timestamp-differs=true

#是否采用只读缓存策略，只读策略对于缓存的数据不会过期。
eureka.server.use-read-only-response-cache=true

```

### 

# Ribbon

ribbon在服务调用过程中起到负载均衡作用。

### 全局配置

ReadTimeout和ConnectTimeout区别是很大的，ConnectTimeout是指建立连接的时间，如果目标服务宕机或网络故障，那么响应的就是ConnectTimeout，无法连接。而ReadTimeout则是连接建立后，等待目标服务返回响应的时间，譬如目标服务做了一个复杂操作导致耗时较长，那么会触发ReadTimeout。

```
ribbon:
  ConnectTimeout: 1000 #服务请求连接超时时间（毫秒）
  ReadTimeout: 3000 #服务请求处理超时时间（毫秒）
  OkToRetryOnAllOperations: true #对超时请求启用重试机制
  MaxAutoRetriesNextServer: 1 #切换重试实例的最大个数
  MaxAutoRetries: 1 # 切换实例后重试最大次数
  NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule #修改负载均衡算法
```


### 指定服务进行配置

指定服务进行配置

与全局配置的区别就是ribbon节点挂在服务名称下面，如下是对produce服务调用时的单独配置。

```
produce:
  ribbon:
    ConnectTimeout: 1000 #服务请求连接超时时间（毫秒）
    ReadTimeout: 3000 #服务请求处理超时时间（毫秒）
    OkToRetryOnAllOperations: true #对超时请求启用重试机制
    MaxAutoRetriesNextServer: 1 #切换重试实例的最大个数
    MaxAutoRetries: 1 # 切换实例后重试最大次数
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule #修改负载均衡算法

```


Ribbon的负载均衡策略

所谓的负载均衡策略，就是当A服务调用B服务时，此时B服务有多个实例，这时A服务以何种方式来选择调用的B实例，ribbon可以选择以下几种负载均衡策略。

com.netflix.loadbalancer.RandomRule：从提供服务的实例中以随机的方式；
com.netflix.loadbalancer.RoundRobinRule：以线性轮询的方式，就是维护一个计数器，从提供服务的实例中按顺序选取，第一次选第一个，第二次选第二个，以此类推，到最后一个以后再从头来过；
com.netflix.loadbalancer.RetryRule：在RoundRobinRule的基础上添加重试机制，即在指定的重试时间内，反复使用线性轮询策略来选择可用实例；
com.netflix.loadbalancer.WeightedResponseTimeRule：对RoundRobinRule的扩展，响应速度越快的实例选择权重越大，越容易被选择；
com.netflix.loadbalancer.BestAvailableRule：选择并发较小的实例；
com.netflix.loadbalancer.AvailabilityFilteringRule：先过滤掉故障实例，再选择并发较小的实例；
com.netflix.loadbalancer.ZoneAwareLoadBalancer：采用双重过滤，同时过滤不是同一区域的实例和故障实例，选择并发较小的实例。

### Ribbon负载均衡源码

ILoadBalancer接口中定义了关于获取服务（Iping）以及选择服务（Irule）的方法，

```
public interface ILoadBalancer {
   //添加一个服务
   public void addServers(List<Server> newServers);
   //负载均衡选择一个服务，调用Irule
   public Server chooseServer(Object key);
   
  //服务下线，如果经过Iping服务的isAlive状态为false，那么服务的状态再次设置为false并通知            notifyServerStatusChangeListener监听器，netfli未实现监听器，如果有需要可以自己实现。
   public void markServerDown(Server server);
   //获取所有的服务，已经过期，使用getReachableServers，相当于availableOnly为true
   public List<Server> getServerList(boolean availableOnly);

    public List<Server> getReachableServers();
    //获取所有的服务  
   public List<Server> getAllServers();
```

默认实现ILoadBalancer接口的是BaseLoadBalancer。

```
   public class BaseLoadBalancer extends AbstractLoadBalancer implements
        PrimeConnections.PrimeConnectionListener, IClientConfigAware {
        
   //变量
   private static Logger logger = LoggerFactory
            .getLogger(BaseLoadBalancer.class);
    //默认的负载均衡策略       
    private final static IRule DEFAULT_RULE = new RoundRobinRule();
    private final static SerialPingStrategy DEFAULT_PING_STRATEGY = new SerialPingStrategy();
    private static final String DEFAULT_NAME = "default";
    private static final String PREFIX = "LoadBalancer_";

    protected IRule rule = DEFAULT_RULE;

    protected IPingStrategy pingStrategy = DEFAULT_PING_STRATEGY;

    protected IPing ping = null;

    @Monitor(name = PREFIX + "AllServerList", type = DataSourceType.INFORMATIONAL)
    protected volatile List<Server> allServerList = Collections
            .synchronizedList(new ArrayList<Server>());
    @Monitor(name = PREFIX + "UpServerList", type = DataSourceType.INFORMATIONAL)
    protected volatile List<Server> upServerList = Collections
            .synchronizedList(new ArrayList<Server>());

    protected ReadWriteLock allServerLock = new ReentrantReadWriteLock();
    protected ReadWriteLock upServerLock = new ReentrantReadWriteLock();

    protected String name = DEFAULT_NAME;

    protected Timer lbTimer = null;
    protected int pingIntervalSeconds = 10;
    protected int maxTotalPingTimeSeconds = 5;
    protected Comparator<Server> serverComparator = new ServerComparator();

    protected AtomicBoolean pingInProgress = new AtomicBoolean(false);

    protected LoadBalancerStats lbStats;

    private volatile Counter counter = Monitors.newCounter("LoadBalancer_ChooseServer");

    private PrimeConnections primeConnections;

    private volatile boolean enablePrimingConnections = false;
    
    private IClientConfig config;
    
    private List<ServerListChangeListener> changeListeners = new CopyOnWriteArrayList<ServerListChangeListener>();

    private List<ServerStatusChangeListener> serverStatusListeners = new CopyOnWriteArrayList<ServerStatusChangeListener>();
    //默认的构造方法
public BaseLoadBalancer() {
    this.name = DEFAULT_NAME;
    this.ping = null;
    //设置默认的负载均衡策略
    setRule(DEFAULT_RULE);
    //定时执行Iping确认服务是否存活，一直跟下去会看到调用Iping的isAlive方法
    setupPingTask();
    lbStats = new LoadBalancerStats(DEFAULT_NAME);
}
```

修改ribbon的负载均衡策略一般可以用来做灰度发布，把新功能的地址记录到一个serverList,redis记录用户ip，下次早进来判断是否已经访问过新功能，如果是直接在serverList里调一个地址访问。

####  RandomRule

```
public Server choose(ILoadBalancer lb, Object key) {
        if (lb == null) {
            return null;
        }
        Server server = null;

        while (server == null) {
            if (Thread.interrupted()) {
                return null;
            }
            在线集合，全部都存活的Server
            List<Server> upList = lb.getReachableServers();
            所有集合，有可能包含未上线或者已下线
            List<Server> allList = lb.getAllServers();

            int serverCount = allList.size();
            if (serverCount == 0) {
                /*
                 * No servers. End regardless of pass, because subsequent passes
                 * only get more restrictive.
                 */
                return null;
            }

            int index = chooseRandomInt(serverCount);
            server = upList.get(index);

            if (server == null) {
                /*
                 * The only time this should happen is if the server list were
                 * somehow trimmed. This is a transient condition. Retry after
                 * yielding.
                 */
                Thread.yield();
                continue;
            }

            if (server.isAlive()) {
                return (server);
            }

            // Shouldn't actually happen.. but must be transient or a bug.
            server = null;
            Thread.yield();
        }

        return server;

    }
```

### IPing

探测服务是否存活

NIWSDiscoveryPing：不执行ping操作，根据EurekaClient的反馈

PingUrl：使用HttpClient对服务进行Ping操作

DummyPing：默认返回true

NoOpPing：永远返回true

# Feign

在为服务中的角色是服务调用

基本配置

```

# 配置请求GZIP压缩
feign.compression.request.enabled=true

# 配置响应GZIP压缩
feign.compression.response.enabled=true

# 配置压缩支持的MIME TYPE
feign.compression.request.mime-types=text/xml,application/xml,application/json

# 配置压缩数据大小的下限
feign.compression.request.min-request-size=2048

```

# Hystrix

### 全局配置

Feign中Hystrix的配置和Ribbon有点像，基础配置如下：

```
#设置熔断超时时间，要大于ribbon的（ReadTimeout+ConnectTimeout）* 重试次数，不然ribbon还在重试，hystrix已经降级
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds=10000

# 关闭Hystrix功能（不要和上面的配置一起使用）
feign.hystrix.enabled=false


# 关闭熔断功能
hystrix.command.default.execution.timeout.enabled=false
```

### 部分配置

如果我们想针对某一个接口配置，比如/hello接口，那么可以按照下面这种写法，如下：

```
# 设置熔断超时时间
hystrix.command.hello.execution.isolation.thread.timeoutInMilliseconds=10000

# 关闭熔断功能
hystrix.command.hello.execution.timeout.enabled=false
```

但是我们的接口名可能会重复，这个时候同名的接口会共用这一条Hystrix配置。

线程隔离

请求缓存

请求合并

# gateway



DynamicServerListLoadBalancer 的updateListOfServers方法进入 会看到从注册中心拉取服务 获取服务 然后设置到 BaseLoadBalancer的serverList中


