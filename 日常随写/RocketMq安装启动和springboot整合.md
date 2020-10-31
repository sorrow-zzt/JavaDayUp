# RocketMq安装启动和springboot整合

## 1.安装RocketMq

### 1.1准备工作

#### 1.1.1安装jdk1.8+和环境变量

```java
export JAVA_HOME=/usr/java/jdk1.8.0_192
export CLASSPATH=$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib
export PATH=$JAVA_HOME/bin
```



#### 1.1.2安装maven3.2.x,并配置环境变量

```java
export MAVEN_HOME=/usr/local/etc/apache-maven-3.6.3
export MAVEN_HOME
export PATH=$PATH:$MAVEN_HOME/bin
```

此处由于maven仓库默认连接的是国外的远程仓库地址,换成国内镜像

```xml
<servers>
	<server>
		<id>releases</id>
		<username>ali</username>
		<password>ali</password>
	  </server>
	  <server>
		<id>Snapshots</id>
		<username>ali</username>
		<password>ali</password>
	  </server>
  </servers>
  <mirrors>
	<mirror>
      <!--This sends everything else to /public -->
      <id>nexus</id>
      <mirrorOf>*</mirrorOf> 
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
    </mirror>
	<mirror>
      <!--This is used to direct the public snapshots repo in the 
          profile below over to a different nexus group -->
      <id>nexus-public-snapshots</id>
      <mirrorOf>public-snapshots</mirrorOf> 
      <url>http://maven.aliyun.com/nexus/content/repositories/snapshots/</url>
    </mirror>
  </mirrors>
```

#### 1.1.3构建成二进制文件

从 <https://www.apache.org/dyn/closer.cgi?path=rocketmq/4.2.0/rocketmq-all-4.2.0-source-release.zip> 下载 4.2.0 的源码版本，执行以下命令来解压4.2.0源码版本并构建二进制文件。

```java
unzip rocketmq-all-4.2.0-source-release.zip

cd rocketmq-all-4.2.0/

mvn -Prelease-all -DskipTests clean install -U
//此处的maven操作需要安装好maven环境和jdk环境
```

构建成功如下:

![1588818852394](C:\Users\lx-PC\AppData\Roaming\Typora\typora-user-images\1588818852394.png)

#### 1.1.4进入主目录并检查内存配置

```java
cd distribution/target/apache-rocketmq
```

如果启动nameserver报内存错误如:

```java
Java HotSpot(TM) 64-Bit Server VM warning: INFO: os::commit_memory(0x00000005c0000000, 8589934592, 0) failed; error='Cannot allocate memory' (errno=12)
```

修改bin/runserver.sh为:

```java
JAVA_OPT="${JAVA_OPT} -server -Xms128m -Xmx256m -Xmn256m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
```

同理启动broker报内存错误修改bin/runbroker.sh为:

```java
JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m"
```

#### 1.1.5通过二进制文件直接使用rocketMq

1. 解压rocketmq-all-4.7.0-bin-release.zip

2. 在rocketmq-all-4.7.0目录下创建store/commitlog和consumequeue和index三个目录

3. 修改conf/broker.conf

   ```xml
   #所属集群名字
   brokerClusterName=rocketmq-cluster
   #broker名字，注意此处不同的配置文件填写的不一样
   brokerName=broker-a
   #0 表示 Master，>0 表示 Slave
   brokerId=0
   brokerIP1=47.92.196.171
   #nameServer地址，分号分割
   namesrvAddr=47.92.196.171:9876
   #在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
   defaultTopicQueueNums=4
   #是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
   autoCreateTopicEnable=false
   #是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
   autoCreateSubscriptionGroup=false
   #Broker 对外服务的监听端口
   listenPort=10911
   #删除文件时间点，默认凌晨 4点
   deleteWhen=04
   #文件保留时间，默认 48 小时
   fileReservedTime=120
   #commitLog每个文件的大小默认1G
   mapedFileSizeCommitLog=1073741824
   #ConsumeQueue每个文件默认存30W条，根据业务情况调整
   mapedFileSizeConsumeQueue=300000
   #destroyMapedFileIntervalForcibly=120000
   #redeleteHangedFileInterval=120000
   #检测物理文件磁盘空间
   diskMaxUsedSpaceRatio=88
   #存储路径
   storePathRootDir=/usr/local/rocketmq-4.7.0/store
   #commitLog 存储路径
   storePathCommitLog=/usr/local/rocketmq-4.7.0/store/commitlog
   #消费队列存储路径存储路径
   storePathConsumeQueue=/usr/local/rocketmq-4.7.0/store/consumequeue
   #消息索引存储路径
   storePathIndex=/usr/local/rocketmq-4.7.0/store/index
   #checkpoint 文件存储路径
   storeCheckpoint=/usr/local/rocketmq-4.7.0/store/checkpoint
   #abort 文件存储路径
   abortFile=/usr/local/rocketmq-4.7.0/store/abort
   #限制的消息大小
   maxMessageSize=65536
   #flushCommitLogLeastPages=4
   #flushConsumeQueueLeastPages=2
   #flushCommitLogThoroughInterval=10000
   #flushConsumeQueueThoroughInterval=60000
   #Broker 的角色
   #- ASYNC_MASTER 异步复制Master
   #- SYNC_MASTER 同步双写Master
   #- SLAVE
   brokerRole=SYNC_MASTER
   #刷盘方式
   #- ASYNC_FLUSH 异步刷盘
   #- SYNC_FLUSH 同步刷盘
   flushDiskType=ASYNC_FLUSH
   #checkTransactionMessageEnable=false
   #发消息线程池数量
   #sendMessageThreadPoolNums=128
   #拉消息线程池数量
   #pullMessageThreadPoolNums=128
   ```

   修改runServer.sh

   ````xml
   JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
   ````

   修改runBroker.sh

   ````xml
   JAVA_OPT="${JAVA_OPT} -server -Xms512m -Xmx512m -Xmn256m"
   ````

4. 分别启动namesrv和broker

   ````bash
   nohup sh mqnamesrv &
   ````

   ````bash
   nohup sh mqbroker -c /usr/local/rocketmq/conf/2m-2s-sync/broker-a.properties &
   ````

   5.开启服务器的9876,10911端口

### 2.1启动nameserver和broker

```java
启动nameserver:
nohup sh mqnamesrv -n 218.106.116.242 9876 &
    
    
    
nohup sh mqnamesrv >/data0/mqlog/rocketmqlogs/mqnamesrv.log 2>&1 &						9876
启动broker:
nohup sh mqbroker -n 218.106.116.242:9876 -c ../conf/broker.conf &
    
    
    
    
autoCreateTopicEnable=true >/usr/local/etc/log/broker.log 2>&1 &
nohup sh mqbroker -n localhost:9876 >/data0/mqlog/rocketmqlogs/broker.log 2>&1 &
    10911
测试发送消息:sh tools.sh org.apache.rocketmq.example.quickstart.Producer
测试接收消息:sh tools.sh org.apache.rocketmq.example.quickstart.Consumer
关闭服务器:	sh mqshutdown broker    //停止 broker
			sh mqshutdown namesrv   //停止 nameserver
查看broker集群服务:sh mqadmin clusterList -n 127.0.0.1:9876
查看 broker 状态 sh mqadmin brokerStatus -n 127.0.0.1:9876 -b 172.26.246.77:10911 (注意换成你的 broker 地址)
查看 topic 列表 sh mqadmin topicList -n 127.0.0.1:9876
查看 topic 状态 sh mqadmin topicStatus -n 127.0.0.1:9876 -t MyTopic (换成你想查询的 topic)
查看 topic 路由 sh mqadmin topicRoute -n 127.0.0.1:9876 -t MyTopic
```

## 2.安装web可视化客户端

### 2.1下载文件

```java
https://github.com/apache/rocketmq-externals 地址下载并从本地上传到服务器
```

### **2.2解压(/usr/local目录下)**

------

yum install -y unzip zip 前提是：unzip解压文件无法使用

unzip rocketmq-externals-master.zip 解压文件

### 2.3.修改配置文件(usr/local/rocketmq-externals-master/rocketmq-console/src/目录下)

------

vim application.properties

rocketmq.config.namesrvAddr=127.0.0.1:9876（ip1:port;ip2:port）

### 2.4.编译(usr/local/rocketmq-externals-master/rocketmq-console/目录下)

------

mvn clean package -Dmaven.test.skip=true 如果失败多编译几次--可能是网络问题

### 2.5.启动web(usr/local/rocketmq-externals-master/rocketmq-console/target目录下)

---

java -jar rocketmq-console-ng-1.0.0.jar 启动 ---当终端断了该服务就会停止

nohup java -jar rocketmq-console-ng-1.0.0.jar >>/soft/RocketMQ/rocketmqlogs/log.out 2>&1 &
后台启动 --当终端断了也不会停止服务

![1588825631729](C:\Users\lx-PC\AppData\Roaming\Typora\typora-user-images\1588825631729.png)



## 3.Linux远程单机部署遇到的问题

### 3.1 内外网ip不一致导致的访问不到namesrv或broker

![1589005700742](C:\Users\lx-PC\AppData\Roaming\Typora\typora-user-images\1589005700742.png)

1. 需要把对应的9876和10909和10911,10912端口放开

2. 在启动namesrv是指定启动地址为公网ip加端口	-n 218.106.116.242 9876

修改broker.conf的配置文件中的brokerIP1=218.106.116.242,然后启动时指定namesrv的公网地址＋配置文件

sh mqbroker -n 218.106.116.242:9876 -c ../conf/broker.conf





## 4.RocketMq基本概念

### 1 消息模型（Message Model）

RocketMQ主要由 Producer、Broker、Consumer 三部分组成，其中Producer 负责生产消息，Consumer 负责消费消息，Broker 负责存储消息。Broker 在实际部署过程中对应一台服务器，每个 Broker 可以存储多个Topic的消息，每个Topic的消息也可以分片存储于不同的 Broker。Message Queue 用于存储消息的物理地址，每个Topic中的消息地址存储于多个 Message Queue 中。ConsumerGroup 由多个Consumer 实例构成。

### 2 消息生产者（Producer）

负责生产消息，一般由业务系统负责生产消息。一个消息生产者会把业务应用系统里产生的消息发送到broker服务器。RocketMQ提供多种发送方式，同步发送、异步发送、顺序发送、单向发送。同步和异步方式均需要Broker返回确认信息，单向发送不需要。

### 3 消息消费者（Consumer）

负责消费消息，一般是后台系统负责异步消费。一个消息消费者会从Broker服务器拉取消息、并将其提供给应用程序。从用户应用的角度而言提供了两种消费形式：拉取式消费、推动式消费。

### 4 主题（Topic）

表示一类消息的集合，每个主题包含若干条消息，每条消息只能属于一个主题，是RocketMQ进行消息订阅的基本单位。

### 5 代理服务器（Broker Server）

消息中转角色，负责存储消息、转发消息。代理服务器在RocketMQ系统中负责接收从生产者发送来的消息并存储、同时为消费者的拉取请求作准备。代理服务器也存储消息相关的元数据，包括消费者组、消费进度偏移和主题和队列消息等。

### 6 名字服务（Name Server）

名称服务充当路由消息的提供者。生产者或消费者能够通过名字服务查找各主题相应的Broker IP列表。多个Namesrv实例组成集群，但相互独立，没有信息交换。

### 7 拉取式消费（Pull Consumer）

Consumer消费的一种类型，应用通常主动调用Consumer的拉消息方法从Broker服务器拉消息、主动权由应用控制。一旦获取了批量消息，应用就会启动消费过程。

### 8 推动式消费（Push Consumer）

Consumer消费的一种类型，该模式下Broker收到数据后会主动推送给消费端，该消费模式一般实时性较高。

### 9 生产者组（Producer Group）

同一类Producer的集合，这类Producer发送同一类消息且发送逻辑一致。如果发送的是事务消息且原始生产者在发送之后崩溃，则Broker服务器会联系同一生产者组的其他生产者实例以提交或回溯消费。

### 10 消费者组（Consumer Group）

同一类Consumer的集合，这类Consumer通常消费同一类消息且消费逻辑一致。消费者组使得在消息消费方面，实现负载均衡和容错的目标变得非常容易。要注意的是，消费者组的消费者实例必须订阅完全相同的Topic。RocketMQ 支持两种消息模式：集群消费（Clustering）和广播消费（Broadcasting）。

### 11 集群消费（Clustering）

集群消费模式下,相同Consumer Group的每个Consumer实例平均分摊消息。

### 12 广播消费（Broadcasting）

广播消费模式下，相同Consumer Group的每个Consumer实例都接收全量的消息。

### 13 普通顺序消息（Normal Ordered Message）

普通顺序消费模式下，消费者通过同一个消费队列收到的消息是有顺序的，不同消息队列收到的消息则可能是无顺序的。

### 14 严格顺序消息（Strictly Ordered Message）

严格顺序消息模式下，消费者收到的所有消息均是有顺序的。

### 15 消息（Message）

消息系统所传输信息的物理载体，生产和消费数据的最小单位，每条消息必须属于一个主题。RocketMQ中每个消息拥有唯一的Message ID，且可以携带具有业务标识的Key。系统提供了通过Message ID和Key查询消息的功能。

### 16 标签（Tag）

为消息设置的标志，用于同一主题下区分不同类型的消息。来自同一业务单元的消息，可以根据不同业务目的在同一主题下设置不同标签。标签能够有效地保持代码的清晰度和连贯性，并优化RocketMQ提供的查询系统。消费者可以根据Tag实现对不同子主题的不同消费逻辑，实现更好的扩展性。

## 5.RocketMq基本特性

### 1 订阅与发布

消息的发布是指某个生产者向某个topic发送消息；消息的订阅是指某个消费者关注了某个topic中带有某些tag的消息，进而从该topic消费数据。

### 2 消息顺序

消息有序指的是一类消息消费时，能按照发送的顺序来消费。例如：一个订单产生了三条消息分别是订单创建、订单付款、订单完成。消费时要按照这个顺序消费才能有意义，但是同时订单之间是可以并行消费的。RocketMQ可以严格的保证消息有序。

顺序消息分为全局顺序消息与分区顺序消息，全局顺序是指某个Topic下的所有消息都要保证顺序；部分顺序消息只要保证每一组消息被顺序消费即可。

- 全局顺序 对于指定的一个 Topic，所有消息按照严格的先入先出（FIFO）的顺序进行发布和消费。 适用场景：性能要求不高，所有的消息严格按照 FIFO 原则进行消息发布和消费的场景
- 分区顺序 对于指定的一个 Topic，所有消息根据 sharding key 进行区块分区。 同一个分区内的消息按照严格的 FIFO 顺序进行发布和消费。 Sharding key 是顺序消息中用来区分不同分区的关键字段，和普通消息的 Key 是完全不同的概念。 适用场景：性能要求高，以 sharding key 作为分区字段，在同一个区块中严格的按照 FIFO 原则进行消息发布和消费的场景。

### 3 消息过滤

RocketMQ的消费者可以根据Tag进行消息过滤，也支持自定义属性过滤。消息过滤目前是在Broker端实现的，优点是减少了对于Consumer无用消息的网络传输，缺点是增加了Broker的负担、而且实现相对复杂。

### 4 消息可靠性

RocketMQ支持消息的高可靠，影响消息可靠性的几种情况：

1. Broker非正常关闭
2. Broker异常Crash
3. OS Crash
4. 机器掉电，但是能立即恢复供电情况
5. 机器无法开机（可能是cpu、主板、内存等关键设备损坏）
6. 磁盘设备损坏

1)、2)、3)、4) 四种情况都属于硬件资源可立即恢复情况，RocketMQ在这四种情况下能保证消息不丢，或者丢失少量数据（依赖刷盘方式是同步还是异步）。

5)、6)属于单点故障，且无法恢复，一旦发生，在此单点上的消息全部丢失。RocketMQ在这两种情况下，通过异步复制，可保证99%的消息不丢，但是仍然会有极少量的消息可能丢失。通过同步双写技术可以完全避免单点，同步双写势必会影响性能，适合对消息可靠性要求极高的场合，例如与Money相关的应用。注：RocketMQ从3.0版本开始支持同步双写。

### 5 至少一次

至少一次(At least Once)指每个消息必须投递一次。Consumer先Pull消息到本地，消费完成后，才向服务器返回ack，如果没有消费一定不会ack消息，所以RocketMQ可以很好的支持此特性。

6 回溯消费

回溯消费是指Consumer已经消费成功的消息，由于业务上需求需要重新消费，要支持此功能，Broker在向Consumer投递成功消息后，消息仍然需要保留。并且重新消费一般是按照时间维度，例如由于Consumer系统故障，恢复后需要重新消费1小时前的数据，那么Broker要提供一种机制，可以按照时间维度来回退消费进度。RocketMQ支持按照时间回溯消费，时间维度精确到毫秒。

### 7 事务消息

RocketMQ事务消息（Transactional Message）是指应用本地事务和发送消息操作可以被定义到全局事务中，要么同时成功，要么同时失败。RocketMQ的事务消息提供类似 X/Open XA 的分布事务功能，通过事务消息能达到分布式事务的最终一致。

### 8 定时消息

定时消息（延迟队列）是指消息发送到broker后，不会立即被消费，等待特定时间投递给真正的topic。 broker有配置项messageDelayLevel，默认值为“1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h”，18个level。可以配置自定义messageDelayLevel。注意，messageDelayLevel是broker的属性，不属于某个topic。发消息时，设置delayLevel等级即可：msg.setDelayLevel(level)。level有以下三种情况：

- level == 0，消息为非延迟消息
- 1<=level<=maxLevel，消息延迟特定时间，例如level==1，延迟1s
- level > maxLevel，则level== maxLevel，例如level==20，延迟2h

定时消息会暂存在名为SCHEDULE_TOPIC_XXXX的topic中，并根据delayTimeLevel存入特定的queue，queueId = delayTimeLevel – 1，即一个queue只存相同延迟的消息，保证具有相同发送延迟的消息能够顺序消费。broker会调度地消费SCHEDULE_TOPIC_XXXX，将消息写入真实的topic。

需要注意的是，定时消息会在第一次写入和调度写入真实topic时都会计数，因此发送数量、tps都会变高。

### 9 消息重试

Consumer消费消息失败后，要提供一种重试机制，令消息再消费一次。Consumer消费消息失败通常可以认为有以下几种情况：

- 由于消息本身的原因，例如反序列化失败，消息数据本身无法处理（例如话费充值，当前消息的手机号被注销，无法充值）等。这种错误通常需要跳过这条消息，再消费其它消息，而这条失败的消息即使立刻重试消费，99%也不成功，所以最好提供一种定时重试机制，即过10秒后再重试。
- 由于依赖的下游应用服务不可用，例如db连接不可用，外系统网络不可达等。遇到这种错误，即使跳过当前失败的消息，消费其他消息同样也会报错。这种情况建议应用sleep 30s，再消费下一条消息，这样可以减轻Broker重试消息的压力。

RocketMQ会为每个消费组都设置一个Topic名称为“%RETRY%+consumerGroup”的重试队列（这里需要注意的是，这个Topic的重试队列是针对消费组，而不是针对每个Topic设置的），用于暂时保存因为各种异常而导致Consumer端无法消费的消息。考虑到异常恢复起来需要一些时间，会为重试队列设置多个重试级别，每个重试级别都有与之对应的重新投递延时，重试次数越多投递延时就越大。RocketMQ对于重试消息的处理是先保存至Topic名称为“SCHEDULE_TOPIC_XXXX”的延迟队列中，后台定时任务按照对应的时间进行Delay后重新保存至“%RETRY%+consumerGroup”的重试队列中。

10 消息重投

生产者在发送消息时，同步消息失败会重投，异步消息有重试，oneway没有任何保证。消息重投保证消息尽可能发送成功、不丢失，但可能会造成消息重复，消息重复在RocketMQ中是无法避免的问题。消息重复在一般情况下不会发生，当出现消息量大、网络抖动，消息重复就会是大概率事件。另外，生产者主动重发、consumer负载变化也会导致重复消息。如下方法可以设置消息重试策略：

- retryTimesWhenSendFailed:同步发送失败重投次数，默认为2，因此生产者会最多尝试发送retryTimesWhenSendFailed + 1次。不会选择上次失败的broker，尝试向其他broker发送，最大程度保证消息不丢。超过重投次数，抛出异常，由客户端保证消息不丢。当出现RemotingException、MQClientException和部分MQBrokerException时会重投。
- retryTimesWhenSendAsyncFailed:异步发送失败重试次数，异步重试不会选择其他broker，仅在同一个broker上做重试，不保证消息不丢。
- retryAnotherBrokerWhenNotStoreOK:消息刷盘（主或备）超时或slave不可用（返回状态非SEND_OK），是否尝试发送到其他broker，默认false。十分重要消息可以开启。

### 11 流量控制

生产者流控，因为broker处理能力达到瓶颈；消费者流控，因为消费能力达到瓶颈。

生产者流控：

- commitLog文件被锁时间超过osPageCacheBusyTimeOutMills时，参数默认为1000ms，返回流控。
- 如果开启transientStorePoolEnable == true，且broker为异步刷盘的主机，且transientStorePool中资源不足，拒绝当前send请求，返回流控。
- broker每隔10ms检查send请求队列头部请求的等待时间，如果超过waitTimeMillsInSendQueue，默认200ms，拒绝当前send请求，返回流控。
- broker通过拒绝send 请求方式实现流量控制。

注意，生产者流控，不会尝试消息重投。

消费者流控：

- 消费者本地缓存消息数超过pullThresholdForQueue时，默认1000。
- 消费者本地缓存消息大小超过pullThresholdSizeForQueue时，默认100MB。
- 消费者本地缓存消息跨度超过consumeConcurrentlyMaxSpan时，默认2000。

消费者流控的结果是降低拉取频率。

### 12 死信队列

死信队列用于处理无法被正常消费的消息。当一条消息初次消费失败，消息队列会自动进行消息重试；达到最大重试次数后，若消费依然失败，则表明消费者在正常情况下无法正确地消费该消息，此时，消息队列 不会立刻将消息丢弃，而是将其发送到该消费者对应的特殊队列中。

RocketMQ将这种正常情况下无法被消费的消息称为死信消息（Dead-Letter Message），将存储死信消息的特殊队列称为死信队列（Dead-Letter Queue）。在RocketMQ中，可以通过使用console控制台对死信队列中的消息进行重发来使得消费者实例再次进行消费。

## 6.整体架构和流程

![1589007930261](C:\Users\lx-PC\AppData\Roaming\Typora\typora-user-images\1589007930261.png)

- 生产者（Producer）：负责产生消息，生产者向消息服务器发送由业务应用程序系统生成的消息。

- 消费者（Consumer）：负责消费消息，消费者从消息服务器拉取信息并将其输入用户应用程序。

- 消息服务器（Broker）：是消息存储中心，主要作用是接收来自 Producer 的消息并存储， Consumer 从这里取得消息。

- 名称服务器（NameServer）：用来保存 Broker 相关 Topic 等元信息并给 Producer ，提供 Consumer 查找 Broker 信息。

  ![1589008010241](C:\Users\lx-PC\AppData\Roaming\Typora\typora-user-images\1589008010241.png)

  - 1、启动 **Namesrv**，Namesrv起 来后监听端口，等待 Broker、Producer、Consumer 连上来，相当于一个路由控制中心。

  - 2、**Broker** 启动，跟所有的 Namesrv 保持长连接，定时发送心跳包。

    > 心跳包中，包含当前 Broker 信息(IP+端口等)以及存储所有 Topic 信息。 注册成功后，Namesrv 集群中就有 Topic 跟 Broker 的映射关系。

    - 3、收发消息前，先创建 Topic 。创建 Topic 时，需要指定该 Topic 要存储在哪些 Broker上。也可以在发送消息时自动创建Topic。

  - 4、**Producer** 发送消息。

    > 启动时，先跟 Namesrv 集群中的其中一台建立长连接，并从Namesrv 中获取当前发送的 Topic 存在哪些 Broker 上，然后跟对应的 Broker 建立长连接，直接向 Broker 发消息。

  - 5、**Consumer** 消费消息。

    > Consumer 跟 Producer 类似。跟其中一台 Namesrv 建立长连接，获取当前订阅 Topic 存在哪些 Broker 上，然后直接跟 Broker 建立连接通道，开始消费消息。


参考:

[1]<http://www.iocoder.cn/RocketMQ/install/>

[2]<http://www.iocoder.cn/RocketMQ/install/#>

[3]<http://www.iocoder.cn/Spring-Boot/RocketMQ/?self>

[4]<https://blog.51cto.com/yushiwh/2118625>

[5]<https://help.aliyun.com/product/29530.html?spm=a2c4g.11186623.6.540.68cc5b3aZYDU2Y>

[6]<https://www.jianshu.com/p/bf96028e8bba>

[7]<https://blog.csdn.net/lw5885799/article/details/88646051>