# 基本概念

##  消息模型（Message Model）

RocketMQ主要由 Producer、Broker、Consumer 三部分组成，其中Producer 负责生产消息，Consumer 负责消费消息，Broker 负责存储消息。Broker 在实际部署过程中对应一台服务器，每个 Broker 可以存储多个Topic的消息，每个Topic的消息也可以分片存储于不同的 Broker。Message Queue 用于存储消息的物理地址，每个Topic中的消息地址存储于多个 Message Queue 中。ConsumerGroup 由多个Consumer 实例构成。

##  消息生产者（Producer）

负责生产消息，一般由业务系统负责生产消息。一个消息生产者会把业务应用系统里产生的消息发送到broker服务器。RocketMQ提供多种发送方式，同步发送、异步发送、顺序发送、单向发送。同步和异步方式均需要Broker返回确认信息，单向发送不需要。

##  消息消费者（Consumer）

负责消费消息，一般是后台系统负责异步消费。一个消息消费者会从Broker服务器拉取消息、并将其提供给应用程序。从用户应用的角度而言提供了两种消费形式：拉取式消费、推动式消费。

##  主题（Topic）

表示一类消息的集合，每个主题包含若干条消息，每条消息只能属于一个主题，是RocketMQ进行消息订阅的基本单位。

##  代理服务器（Broker Server）

消息中转角色，负责存储消息、转发消息。代理服务器在RocketMQ系统中负责接收从生产者发送来的消息并存储、同时为消费者的拉取请求作准备。代理服务器也存储消息相关的元数据，包括消费者组、消费进度偏移和主题和队列消息等。

##  名字服务（Name Server）

名称服务充当路由消息的提供者。生产者或消费者能够通过名字服务查找各主题相应的Broker IP列表。多个Namesrv实例组成集群，但相互独立，没有信息交换。

##  拉取式消费（Pull Consumer）

Consumer消费的一种类型，应用通常主动调用Consumer的拉消息方法从Broker服务器拉消息、主动权由应用控制。一旦获取了批量消息，应用就会启动消费过程。

##  推动式消费（Push Consumer）

Consumer消费的一种类型，该模式下Broker收到数据后会主动推送给消费端，该消费模式一般实时性较高。

##  生产者组（Producer Group）

同一类Producer的集合，这类Producer发送同一类消息且发送逻辑一致。如果发送的是事务消息且原始生产者在发送之后崩溃，则Broker服务器会联系同一生产者组的其他生产者实例以提交或回溯消费。

##  消费者组（Consumer Group）

同一类Consumer的集合，这类Consumer通常消费同一类消息且消费逻辑一致。消费者组使得在消息消费方面，实现负载均衡和容错的目标变得非常容易。要注意的是，消费者组的消费者实例必须订阅完全相同的Topic。RocketMQ 支持两种消息模式：集群消费（Clustering）和广播消费（Broadcasting）。

**注意点**

- 如果两个消费订阅了同一个topic**但是有不同的 Consumer  Group名称**，则这两个消费者组都会收到消息（即使是集群模式），因为broker记录了两个消费者组的消费偏移量

```
"PubSubscribeTopic@PubSubscribeConsume":{0:0,1:3,2:1,3:0 
},

"PubSubscribeTopic@PubSubscribeConsume1":{0:0,1:3,2:1,3:0
},
```

- 如果两个消费者Consumer  Group名称一样，那么两个消费者只会有一个收到消息（集群模式下）

##  集群消费（Clustering）

集群消费模式下,相同Consumer Group的每个Consumer实例平均分摊消息。

##  广播消费（Broadcasting）

广播消费模式下，相同Consumer Group的每个Consumer实例都接收全量的消息。

##  普通顺序消息（Normal Ordered Message）

普通顺序消费模式下，消费者通过同一个消费队列收到的消息是有顺序的，不同消息队列收到的消息则可能是无顺序的。

##  严格顺序消息（Strictly Ordered Message）

严格顺序消息模式下，消费者收到的所有消息均是有顺序的。

##  消息（Message）

消息系统所传输信息的物理载体，生产和消费数据的最小单位，每条消息必须属于一个主题。RocketMQ中每个消息拥有唯一的Message ID，且可以携带具有业务标识的Key。系统提供了通过Message ID和Key查询消息的功能。

##  标签（Tag）

为消息设置的标志，用于同一主题下区分不同类型的消息。来自同一业务单元的消息，可以根据不同业务目的在同一主题下设置不同标签。标签能够有效地保持代码的清晰度和连贯性，并优化RocketMQ提供的查询系统。消费者可以根据Tag实现对不同子主题的不同消费逻辑，实现更好的扩展性。
# 消息存储

## commitLog

存储消息 顺序写 随机读,broker上的所有topic共用，也就是说，一台broker上的所有消息都会顺序写再commitlog上，一个commitlog文件大小为1G

```
public class CommitLog {
  
    protected final MappedFileQueue mappedFileQueue;
    
    protected final DefaultMessageStore defaultMessageStore;
    
    private final FlushCommitLogService flushCommitLogService;
    }
```

![74433151c88f33be9220ff5ba9d5e8f](C:\Users\wql\Desktop\74433151c88f33be9220ff5ba9d5e8f.png)

## MappedFileQueue

```
public class MappedFileQueue {
 

    private final String storePath;

    private final int mappedFileSize;

    private final CopyOnWriteArrayList<MappedFile> mappedFiles = new CopyOnWriteArrayList<MappedFile>();
    }
```

## MappedFile

```
public class MappedFile extends ReferenceResource {
    public static final int OS_PAGE_SIZE = 1024 * 4;
  

    private static final AtomicInteger TOTAL_MAPPED_FILES = new AtomicInteger(0);
    protected final AtomicInteger wrotePosition = new AtomicInteger(0);
    protected final AtomicInteger committedPosition = new AtomicInteger(0);
    private final AtomicInteger flushedPosition = new AtomicInteger(0);
    protected int fileSize;
    protected FileChannel fileChannel;
   
    protected ByteBuffer writeBuffer = null;
    protected TransientStorePool transientStorePool = null;
    private String fileName;
    private long fileFromOffset;
    private File file;
    private MappedByteBuffer mappedByteBuffer;
    private volatile long storeTimestamp = 0;
    private boolean firstCreateInQueue = false;
```

## consumeQueue

存储消息在commitLog文件的偏移量，以便在comitlog快速取出数据。一个topic的队列分布在多台broker上，如果此台broker上 testTopic只有queue1 ，那么consumeQueue只存储testTopic下的queue1的消息偏移量（一个queue不会分布在多台broker上）

### consumerOffset

存储消息消费偏移量，存储了Topic下每个队列消息消费的偏移量，  **集群模式**此文件存储在broker端，**广播模式**下存储在消费端；文件内容如下

```
{
	"offsetTable":{
		"%RETRY%EsPosBillItemConsumer@EsPosBillItemConsumer":{0:154
		},
		"datacenter-pos@EsPosBillItemConsumer":{0:616598,1:616598,2:616596,3:616597 
		},
		"datacenter-pos@EsPosBillConsumer":{0:616598,1:616598,2:616596,3:616597
		},
		"%RETRY%EsPosBillConsumer@EsPosBillConsumer":{0:0
		}
	}
}
```



## 什么时候清理物理消息文件？
那消息文件到底删不删，什么时候删？

消息存储在CommitLog之后，的确是会被清理的，但是这个清理只会在以下任一条件成立才会批量删除消息文件（CommitLog）：

消息文件过期（默认48小时），且到达清理时点（默认是凌晨4点），删除过期文件。
消息文件过期（默认48小时），且磁盘空间达到了水位线（默认75%），删除过期文件。
若磁盘已经达到必须释放的上限（85%水位线）的时候，则立即开始批量清理文件（无论是否过期），直到空间充足。但不会拒绝新消息写入
若磁盘空间达到危险水位线（默认90%），出于保护自身的目的，broker会拒绝写入服务并立即清理过期文件
源码：DefaultMessageStore类的deleteExpiredFiles方法

            private void deleteExpiredFiles() {
                int deleteCount = 0;
                long fileReservedTime = DefaultMessageStore.this.getMessageStoreConfig().getFileReservedTime();
                int deletePhysicFilesInterval = DefaultMessageStore.this.getMessageStoreConfig().getDeleteCommitLogFilesInterval();
                int destroyMapedFileIntervalForcibly = DefaultMessageStore.this.getMessageStoreConfig().getDestroyMapedFileIntervalForcibly();
    
                boolean timeup = this.isTimeToDelete();
                boolean spacefull = this.isSpaceToDelete();
                boolean manualDelete = this.manualDeleteFileSeveralTimes > 0;
    
                if (timeup || spacefull || manualDelete) {
    
                    if (manualDelete)
                        this.manualDeleteFileSeveralTimes--;
    
                    boolean cleanAtOnce = DefaultMessageStore.this.getMessageStoreConfig().isCleanFileForciblyEnable() && this.cleanImmediately;
    
                    log.info("begin to delete before {} hours file. timeup: {} spacefull: {} manualDeleteFileSeveralTimes: {} cleanAtOnce: {}",
                        fileReservedTime,
                        timeup,
                        spacefull,
                        manualDeleteFileSeveralTimes,
                        cleanAtOnce);
    
                    fileReservedTime *= 60 * 60 * 1000;
    
                    deleteCount = DefaultMessageStore.this.commitLog.deleteExpiredFile(fileReservedTime, deletePhysicFilesInterval,
                        destroyMapedFileIntervalForcibly, cleanAtOnce);
                    if (deleteCount > 0) {
                    } else if (spacefull) {
                        log.warn("disk space will be full soon, but delete file failed.");
                    }
                }
            }

isSpaceToDetele

```
private boolean isSpaceToDelete() {
            double ratio = DefaultMessageStore.this.getMessageStoreConfig().getDiskMaxUsedSpaceRatio() / 100.0;

            cleanImmediately = false;

            {
                String storePathPhysic = DefaultMessageStore.this.getMessageStoreConfig().getStorePathCommitLog();
                double physicRatio = UtilAll.getDiskPartitionSpaceUsedPercent(storePathPhysic);
                if (physicRatio > diskSpaceWarningLevelRatio) {
                    boolean diskok = DefaultMessageStore.this.runningFlags.getAndMakeDiskFull();
                    if (diskok) {
                        DefaultMessageStore.log.error("physic disk maybe full soon " + physicRatio + ", so mark disk full");
                    }
             若磁盘空间达到危险水位线（默认90%），出于保护自身的目的，broker会拒绝写入服务并立即清理过期文件
                    cleanImmediately = true;
                } else if (physicRatio > diskSpaceCleanForciblyRatio) {
                若磁盘已经达到必须释放的上限（85%水位线）的时候，则立即开始批量清理文件（无论是否过期），直到空间充足。但不会拒绝新消息写入
                    cleanImmediately = true;
                } else {
                    boolean diskok = DefaultMessageStore.this.runningFlags.getAndMakeDiskOK();
                    if (!diskok) {
                        DefaultMessageStore.log.info("physic disk space OK " + physicRatio + ", so mark disk ok");
                    }
                }

                if (physicRatio < 0 || physicRatio > ratio) {
                //磁盘空间达到了水位线（默认75%），删除过期文件。
                    DefaultMessageStore.log.info("physic disk maybe full soon, so reclaim space, " + physicRatio);
                    return true;
                }
            }

            {
                String storePathLogics = StorePathConfigHelper
                    .getStorePathConsumeQueue(DefaultMessageStore.this.getMessageStoreConfig().getStorePathRootDir());
                double logicsRatio = UtilAll.getDiskPartitionSpaceUsedPercent(storePathLogics);
                if (logicsRatio > diskSpaceWarningLevelRatio) {
                    boolean diskok = DefaultMessageStore.this.runningFlags.getAndMakeDiskFull();
                    if (diskok) {
                        DefaultMessageStore.log.error("logics disk maybe full soon " + logicsRatio + ", so mark disk full");
                    }

                    cleanImmediately = true;
                } else if (logicsRatio > diskSpaceCleanForciblyRatio) {
                    cleanImmediately = true;
                } else {
                    boolean diskok = DefaultMessageStore.this.runningFlags.getAndMakeDiskOK();
                    if (!diskok) {
                        DefaultMessageStore.log.info("logics disk space OK " + logicsRatio + ", so mark disk ok");
                    }
                }

                if (logicsRatio < 0 || logicsRatio > ratio) {
                    DefaultMessageStore.log.info("logics disk maybe full soon, so reclaim space, " + logicsRatio);
                    return true;
                }
            }

            return false;
        }
```

# 消息丢失、重复投递，重复消费

在rocketmq中有三种情况消息丢失，分别是生产者向broker发送消息，第二种是broker存储消息，第三种是消费者消费消息。

## 生产端发送消息到broker

- 生产端发送消息，可能在produce--------》borker过程网络闪断导致**消息丢失**，解决：采用rocketmq sendResult并且消息重试机制，可以根据produce sendResult 状态进行处理
- broker-------》produce 回复网络闪断，消息已经到达了broker，但是produce没有收到sendResult ，则会进行消息重试导致**消息重复投递**（此过程不会导致消息丢失）

## broker存储消息

broker收到produce的消息是先放在内存中，如果是同步刷盘，则不会出现消息丢失，因为消息到达broker就会进行持久化，如果是异步刷盘则会导致**消息丢失**

## 消费者消费消息

消费者消费消息后会给broker一个ack。如果ack在网络闪断形况下丢失。

consume-------------》broker  网络闪断丢失ack，则broker不会修改consumeOffset的消费消息偏移量，则会造成**消息重复消费**

## 消息丢失解决方案

produce使用sendResult 并且开始消息重试，如果单机模式broker开始同步刷盘，集群模式为了保证吞吐量可以采用异步刷盘，同步复制

## 消息重复投递解决方案

保证消息可靠传递就不能保证消息重复投递

## 消息重复消费解决方案

消费端幂等性校验