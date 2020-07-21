---
description: rocket mq相关学习笔记
---

# rocketMQ部分

## rocketMQ都有哪些部分组成

rocketMQ是一个消息中间件，它由nameserver、broker、producer、consumer四个部分组成。

* nameserver是一个名字服务中心，主要存储broker以及topic相关信息，nameserver会与broker建立长连接，broker会30秒像nameserver发送心跳包，心跳包中附带的是topic相关的信息。
* broker代理服务器，它是一个消息中转角色，负责消息的存储以及转发消息，broker在rocketMQ系统中负责接收生产者发送来的消息并存储，同时为消费者的拉取请求做准备，如果消息消费方式是推送方式broker接收到消息后会主动推送消息至消费端，broker也存储相关的元数据，包括消费组，消费进度偏移和主题队列消息。
* producer生产者，生产者则是生成消息的角色，producer发送消息的时候会先去nameserver获取topic对应的broker，然后将消息发送至broker，producer发送消息的方式有同步发送、异步发送、顺序发送、单向发送，同步发送和异步发送均需要broker返回确认信息，单向发送则不需要。
* consumer消费者，负责消费消息，从broker服务器拉取消息，或者是broker推送至消息消费者。

## 请说说你对nameserver的理解。

nameserver名字服务器，它主要是存储broker以及topic相关的数据，与每一个broker保持着长连接，broker会定时每30s像nameserver发送心跳包，该心跳包包含broker所负责的所有topic相关信息以及broker本身的信息，所以broker即使在某个时间点宕机了，nameserver也不会立即知道，而且nameserver之间的信息是不存在互通的，从这一点可以看出nameserver的设计理念是弱化了一致性原则，如果存在消息发送到宕机的broker则是通过重试来解决，这样的设计主要是为了简化nameserver的复杂度，也减轻了nameserver的压力，nameserver的主要性能开销还是在于心跳包的维护。

## 请说说broker消息储存机制。

broker消息储存主要有三个部分组成。

1. CommitLog：消息主体以及元数据的存储主体，存储Producer端写入的消息主体内容,消息内容不是定长的。单个文件大小默认1G ，文件名长度为20位，左边补零，剩余为起始偏移量，比如00000000000000000000代表了第一个文件，起始偏移量为0，文件大小为1G=1073741824；当第一个文件写满了，第二个文件为00000000001073741824，起始偏移量为1073741824，以此类推。消息主要是顺序写入日志文件，当文件满了，写入下一个文件；
2. ConsumeQueue：消息消费队列，引入的目的主要是提高消息消费的性能，由于RocketMQ是基于主题topic的订阅模式，消息消费是针对主题进行的，如果要遍历commitlog文件中根据topic检索消息是非常低效的。Consumer即可根据ConsumeQueue来查找待消费的消息。其中，ConsumeQueue（逻辑消费队列）作为消费消息的索引，保存了指定Topic下的队列消息在CommitLog中的起始物理偏移量offset，消息大小size和消息Tag的HashCode值。consumequeue文件可以看成是基于topic的commitlog索引文件，故consumequeue文件夹的组织方式如下：topic/queue/file三层组织结构，具体存储路径为：$HOME/store/consumequeue/{topic}/{queueId}/{fileName}。同样consumequeue文件采取定长设计，每一个条目共20个字节，分别为8字节的commitlog物理偏移量、4字节的消息长度、8字节tag hashcode，单个文件由30W个条目组成，可以像数组一样随机访问每一个条目，每个ConsumeQueue文件大小约5.72M；
3. IndexFile：IndexFile（索引文件）提供了一种可以通过key或时间区间来查询消息的方法。Index文件的存储位置是：$HOME \store\index${fileName}，文件名fileName是以创建时的时间戳命名的，固定的单个IndexFile文件大小约为400M，一个IndexFile可以保存 2000W个索引，IndexFile的底层存储设计为在文件系统中实现HashMap结构，故rocketmq的索引文件其底层实现为hash索引。

## rocketMQ如何通过tag做消息过滤？

rocketMQ的消息过滤是坐在客户端的，每个客户端消费消息都需要通过consumeQueue逻辑队列拿到一个索引，然后再通过索引去CommitLog文件中取实体消息，消息过滤的逻辑是通过CommitLog中存储的tag hashcode值来进行过滤。

过滤过程： consumer在订阅topic的时候可以指定tag，consumer会将订阅请求构建成一个SubscriptionData，发送一个Pull消息的请求给Broker端。Broker读取数据之前会用这些数据先构建一个MessageFilter，ConsumeQueue读取到一条记录后，会用它记录的消息tag hash值去做过滤，由于在服务端只是根据hashcode进行判断，无法精确对tag原始字符串进行过滤，故在消息消费端拉取到消息后，还需要对消息的原始tag字符串进行比对，如果不同，则丢弃该消息，不进行消息消费。

## rocketMQ如何做负载均衡？

rocketMQ负载均衡主要分为两部分，producer负载均衡、consumer负载均衡。

1. producer负载均衡

Producer端在发送消息的时候，会先根据Topic找到指定的TopicPublishInfo，在获取了TopicPublishInfo路由信息后，RocketMQ的客户端在默认方式下selectOneMessageQueue\(\)方法会从TopicPublishInfo中的messageQueueList中选择一个队列（MessageQueue）进行发送消息。默认的方式为随机递增取模，这里有一个sendLatencyFaultEnable开关变量，如果开启的话会在随机递增取模的基础上在过滤掉not available的Broker代理，所谓的"latencyFaultTolerance"，是指对之前失败的，按一定的时间做退避。退避规则在MQFaultStrategy类中的latencyMax、notAvailableDuration有定义，例如上一次发送消息时间超过550ms则会退避30s才能进行再次消息发送。

1. consumer负载均衡

在RocketMQ中，Consumer端的两种消费模式（Push/Pull）都是基于拉模式来获取消息的，而在Push模式只是对pull模式的一种封装，其本质实现为消息拉取线程在从服务器拉取到一批消息后，然后提交到消息消费线程池后，又“马不停蹄”的继续向服务器再次尝试拉取消息。如果未拉取到消息，则延迟一下又继续拉取。在两种基于拉模式的消费方式（Push/Pull）中，均需要Consumer端在知道从Broker端的哪一个消息队列—队列中去获取消息。因此，有必要在Consumer端来做负载均衡，即Broker端中多个MessageQueue分配给同一个ConsumerGroup中的哪些Consumer消费。

1. Consumer端的心跳包发送

在Consumer启动后，它就会通过定时任务不断地向RocketMQ集群中的所有Broker实例发送心跳包（其中包含了，消息消费分组名称、订阅关系集合、消息通信模式和客户端id的值等信息）。Broker端在收到Consumer的心跳消息后，会将它维护在ConsumerManager的本地缓存变量—consumerTable，同时并将封装后的客户端网络通道信息保存在本地缓存变量—channelInfoTable中，为之后做Consumer端的负载均衡提供可以依据的元数据信息。

1. Consumer端实现负载均衡的核心类—RebalanceImpl

在Consumer实例的启动流程中的启动MQClientInstance实例部分，会完成负载均衡服务线程—RebalanceService的启动（每隔20s执行一次）。通过查看源码可以发现，RebalanceService线程的run\(\)方法最终调用的是RebalanceImpl类的rebalanceByTopic\(\)方法，该方法是实现Consumer端负载均衡的核心。这里，rebalanceByTopic\(\)方法会根据消费者通信类型为“广播模式”还是“集群模式”做不同的逻辑处理。这里主要来看下集群模式下的主要处理流程：

\(1\) 从rebalanceImpl实例的本地缓存变量—topicSubscribeInfoTable中，获取该Topic主题下的消息消费队列集合（mqSet）；

\(2\) 根据topic和consumerGroup为参数调用mQClientFactory.findConsumerIdList\(\)方法向Broker端发送获取该消费组下消费者Id列表的RPC通信请求（Broker端基于前面Consumer端上报的心跳包数据而构建的consumerTable做出响应返回，业务请求码：GET\_CONSUMER\_LIST\_BY\_GROUP）；

\(3\) 先对Topic下的消息消费队列、消费者Id排序，然后用消息队列分配策略算法（默认为：消息队列的平均分配算法），计算出待拉取的消息队列。这里的平均分配算法，类似于分页的算法，将所有MessageQueue排好序类似于记录，将所有消费端Consumer排好序类似页数，并求出每一页需要包含的平均size和每个页面记录的范围range，最后遍历整个range而计算出当前Consumer端应该分配到的记录（这里即为：MessageQueue）。

## rocketMQ 如何实现延时消息？

延时消息，rocketMQ不支持任何时间的延时，只支持特定的延时级别，目前有18个延时级别分别为1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h，rocketMQ内部设置了一个延时topic，名称为SCHEDULE\_TOPIC\_XXXX，所有延时消息发送到broker，broker都会对其消息进行topic替换操作，换成SCHEDULE\_TOPIC\_XXXX，broker本身启动的时候会启动对应的延时消息调度器ScheduleMessageService，每个级别对应一个ScheduleMessageService，所以broker内部启动了18个ScheduleMessageService，每隔特定时间根据offset从消息消费队列中获取当前队列中所有有效的消息。如果未找到，则更新一下延迟队列定时拉取进度并创建定时任务待下一次继续尝试。如果找到可用的消息则对其进行置换，转换为真实的topic再次写入commitLog中，以便于正常业务逻辑消费。

## rocketMQ 如何实现消息消费失败重试？

1. Consumer 消费失败，将消息发送回 Broker 。

Consumer 消息失败后，会将消息的 Topic 修改为 %RETRY% + Topic 进行，添加 "RETRY\_TOPIC" 属性为原始 Topic ，然后再返回给 Broker 中。

1. Broker 收到重试消息之后置换 Topic ，存储消息。

Broker 收到重试消息之后，会有两次修改消息的 Topic 。首先，会将消息的 Topic 修改为 %RETRY% + ConsumerGroup ，因为这个消息是当前消费这分组消费失败，只能被这个消费组所重新消费。消费者会默认订阅 Topic 为 %RETRY% + ConsumerGroup 的消息。然后，会将消息的 Topic 修改为 SCHEDULE\_TOPIC\_XXXX ，添加 "REAL\_TOPIC" 属性为 %RETRY% + ConsumerGroup ，因为重试消息需要延迟消费。

1. Consumer 会拉取该 Topic 对应的 retryTopic 的消息。

Consumer 会拉取该 Topic 对应的 retryTopic 的消息，此处的 retryTopic 为 %RETRY% + ConsumerGroup 。

1. Consumer 拉取到 retryTopic 消息之后，置换到原始的 Topic ，把消息交给 Listener 消费。

Consumer 拉取到 retryTopic 消息之后，置换到原始的 Topic ，因为有消息的 "RETRY\_TOPIC" 属性是原始 Topic ，然后把消息交给 Listener 消费。

> 注意消息消费失败最多16次，消息重试从10s延迟级别开始，16次消息消费失败后会储存到TOPIC名为"%DLQ%" + ConsumerGroup，可以写一个对应的订阅关系进行异常消息警报。

## rocketMQ 如何实现事物消息？

rocketMQ的事物消息也是基于Topic替换的思想。

1.首先Producer发送事物消息至broker，broker会将消息储存到名为Half的topic中，随后会开启一个定时任务进行查询，防止Producer挂掉事物无法确定，需要注意查询最多查询15次，超过15次该消息默认回滚掉。

2.Producer进行事物提交或者回滚的时候，broker会添加一条消息至OP\_Half的topic中，储存的是Half消息对应的offset表示这个事物消息已经提交或者回滚。

3.Producer提交消息的时候，broker会将消息储存至原Topic当中，从而使得对应的consumer进行拉取消费。

## Producer 发送消息有几种方式？

1. 同步方式

同步发送消息，等待broker存储消息结果，成功或者失败当场直接返回。

1. 异步方式

异步发送消息，发送消息的时候会带着回调钩子函数，broker存储消息完毕之后触发回调钩子函数，发送方处理后续业务逻辑。

1. Oneway方式

只管发，不管结果。

## consumer 消费消息过程？

rocketMQ消息消费都是采用的拉模式，没有推模式，虽然推模式的配置方式，但是rocketMQ底层也是采用的拉模式去做，正常拉模式消费流程不用说，就是客户端自己写业务逻辑定时去消息储存中心broker拉取消息，那么包装过一层的推模式到底怎么实现的呢？

首先rocketMQ每个消费者端启动的时候都会带有重启一个PullMessageService服务，该服务会循环从pullRequestQueue队列当中去获取pullRequest对象，pullRequestQueue是一个并发阻塞队列，获取PullRequest对象中的消息处理队列，如果消息处理队列消息堆积过多进行消息流控业务处理，然后获取对应的主题信息以及broker信息，最后把拉取请求发送给服务端，服务端收到请求后会默认阻塞15s的时间，这个阻塞时间是在消息获取不到的时候，如果获取到消息立马进行返回，这样也起到了一个控制作用，如果topic下的队列不存在可消费消息的时候避免过多的无用请求，服务端返回拉取结果后对结果进行解码执行PullCallback回调逻辑，执行消息过滤逻辑后将拉取的消息提交给消费者线程池进行消费。

pullRequestQueue：是一个阻塞队列，消费端自产自销。

PullRequest：这个PullRequest是一个非常重要的对象，他内部封装了消费组、待拉取消息的消费队列、消息处理队列（从broker拉取的消息先入这个队）、待拉取消息的偏移量、锁标志。这个pullRequest对象生成方式有两种，1.消费端的负载均衡每20s会重新进行消费队列负载，每次重新负载之后会生成PullRequest放入pullRequestQueue。2.每次拉取消息之后，只要不是异常都会再次投递进行下次消息拉取。如果拉取消息异常则会延迟一会儿进行下次拉取（默认1s）。

## RocketMQ 如何实现高可用？

1. **NameServer**

NameServer需要部署多个节点，以保证NameServer的高可用，NameServer集群并互相通信，但是可以避免某一个NameServer出现问题的时候导致rocketMQ不可工作。

1. **broker**

多个Broker可以形成一个Broker分组。每个Broker分组存在一个Master和多个Slave节点，Master节点提供读和写功能，Slave 节点提供读功能，Master节点会不断发送新的CommitLog给Slave节点，Slave节点不断上报本地的CommitLog已经同步到的位置给Master 节点。当某个时刻Master出现问题的时候Salve也可以顶上达到高可用效果。

## RocketMQ 是否会弄丢数据？如何避免。

* 消费端：首先消费端不存在丢失消息，因为每个消息消费需要提交一个ack给broker，如果broker没有收到对应的ack表示该消息没有消费，会继续投递给消费组的其他消费者。只是消费端会存在一个重复消费的场景，所以消费端需要做好业务幂等性，防止重复消费对业务造成影响。
* broker节点：什么情况下broker会存在消息丢失了，broker在接受到消息的时候会先存入commitLog中，这个写入不是直接写入磁盘是先写入pageCache中，然后再通过刷盘机制刷到磁盘中，默认配置是异步刷盘，如果是异步刷盘那就存在消息丢失的可能，可以改为同步刷盘以防止消息丢失，但是这样会带来性能的损耗。

## RocketMQ 如何实现顺序消息？

Producer：如果对消息有顺序要求，那么Producer发送消息的时候一定要单线程发送，如果是并发发送消息的话，消息本身进入commitLog就不是顺序，这个时候消费者即便是指定顺序消费也没有意义。

Consumer：消费者，消费者可以实现局部顺序消费，如果需要全局顺序消费则需要配置Topic只有一个逻辑队列与之对应，如果使用阿里云的rocketMQ则只需要指定shardingKey即可，通过shardingKey可以路由到特定的队列下。Consumer局部顺序消费是通过锁来实现的，这个局部指的是消费者对应的逻辑队列部分。

顺序消息与并发消息的区别：1.顺序消息在创建消息队列拉取任务时需要在Broker服务器锁定该消息队列。

