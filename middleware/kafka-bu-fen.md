# kafka部分

### Kafka版本号的特性

- 0.7版本：只实现了最基础的消息队列功能。
- 0.8版本：引入了副本机制，使得Kafka具有高可用性。
- 0.9.0.0版本：增加了基础的安全校验功能，并且采用java重新了新版本的C消费端API，引入了Kafka Connect组件。
- 0.10.0.0版本：引入了Streams流处理，正式变为流处理中间件。
- 0.11.0.0版本：提供了幂等性、事务API，对Kafka消息格式做了重构。
- 1.0和2.0版本：主要还是对Streams进行了改进。

### Kafka的Broker比较重要的参数配置
log.dirs：指定Broker需要使用的文件目录，是多个文件目录。

log.dir：指定Broker需要使用的单个文件目录，一般来说使用log.dirs就可以，log.dirs相较于log.dir具有高性能（采用一个服务器多块磁盘肯定比单块磁盘新能高）、高可用性（如果一块磁盘出现问题，还可以用其他磁盘，这个功能是1.1版本引入的强大的功能）。

auto.create.topics.enable：是否允许自动创建Topic，线上一般建议为false，毕竟自动创建会因为代码问题出现奇奇怪怪的Topic。

unclean.leader.election.enable：是否允许数据同步比较慢的副本选举Leader，一般也建议关闭，Kafka的每个分区都会有副本，这些副本中只有一个Leader副本会对外提供服务，如果当数据同步比较快的副本都挂掉的时候，这个时候如果这个参数设置为false的话这个分区将不可用。

auto.leader.rebalance.enable：是否允许定时切换Leader，这个一般建议禁用，因为切换Leader也是需要代价的，本来好好服务的Leader被切换了，这个时候所有的写服务都需要转换新的连接。


log.retention.{hours|minutes|ms}：这个是消息的储存时间，一般建议都设置为hours，ms级别是最高的，不过不建议用。

log.retention.bytes：Broker所能使用的最大容量，默认是-1，也就是无限制。

message.max.bytes：控制Broker能接受最大单条消息的容量，默认为1Mb，建议改大一点，毕竟这个参数不占用空间，只是一个尺度，但是也不建议无限大。

### Kafka的Topic比较重要的参数配置
retention.ms：规定Topic中储存消息的最长时间，默认是7天，这个参数会覆盖掉Broker上的消息储存时间。

retention.bytes：Topic能使用的最大空间，默认-1是无限制的。

max.message.bytes：Broker能接受该Topic最大消息大小，全局可以设置的很大，但是这个Topic级别上的可以稍微设置的严谨一点，毕竟每个Topic的业务属性都知道了。

### Kafka分区策略以及分区原理

Kafka的消息存储架构为Topic-partition-message，每个Topic下可以有1到N个分区，分区的大小是决定Kafka吞吐量的关键，每个消息只会存在一个分区中，但是每个分区可以有N个副本，N个副本中有一个Leader副本可以提供服务，其他的副本只负责同步数据，这也是Kafka实现高可用的关键。

**Kafka选择分区策略**

Kafka选择分区策略，这里选择分区指的是Producer发送消息的时候，目前分区策略有以下几种：
1. 轮询策略
2. 对消息Key进行HashCode然后再取模
3. 随机策略

Kafka的Producer使用的Partition默认实现了两个策略一个是轮询一个是Key分区，如果消息指定了Key则进行Key分区，如果没有指定则进行轮询策略。


### Kafka Producer端

Producer是线程安全的，如果要增加性能的话需要创建多个Producer。

#### Kafka Producer发送原理逻辑
[kafka Producer原理图](https://www.processon.com/view/link/5fc78d82f346fb646a66d073)

Kafka Producer有两个线程协调运行，一个主线程和一个Sender线程，在主线程中由Producer创建消息，然后可能会通过拦截器、序列化器、分区器，之后消息进入到消息累加器，消息储存在累加器中的ProducerBatch中，按照分区进行储存，Sender线程会从累加器中获取数据然后发送至Kafka的Broker节点上，发送消息之前还会将消息缓存在InFlightRequests对象中，这个对象主要作用就是缓存哪些发送出去的请求但是还未得到回应的连接数据，之后通过回调处理后续的操作。


**注意点：消息累加器有缓存大小限制，buffer.memory参数可以配置，默认是32MB，如果消息累加器消息储存满了那么消息会被阻塞，阻塞时间默认是60S，超过时间就会抛出异常**

**ProducerBatch内部也有内存复用情况，ProducerBatch用固定大小的ByteBuffer做内存缓存池，如果消息发送的比较大那么就无法复用ByteBuffer对象**

**InFlightRequests对象最大连接数为5个，超过5个之后将不会发送数据，需要等待，max.in.flight.requests.per.connection为配置项**

**Producer重要的参数**
1. acks：1-消息发送之后，只要分区的leader副本写入成功就会标记为成功；0-发送之后不需要等待成功消息的响应；-1 or all-消息发送之后，需要等待ISR中所有的副本写入成功才会标记为成功；
2. max.request.size：限制Producer发送的消息最大值，默认是1MB。
3. retries和retry.backoff.ms：这个是用来控制Producer重试的参数，retries为重试次数，retry.backoff.ms为重试间隔。
4. compression.type：Producer发送消息采用的压缩算法，消息经过压缩之后会更加节省内存，默认为none。
5. linger.ms：Producer发送ProducerBatch之前需要等待消息（ProducerRecord）加入到ProducerBatch中，直到被填满或者超过linger.ms设置的值之后再发出去，默认为0，设置这个数可以增大吞吐量，但是也会增加消息的延迟性。
6. request.timeout.ms：Producer发送消息等待的最长时间，默认为30S。

#### Kafka Producer如何管理TCP连接
众所周知Kafka生产者发送消息是基于TCP连接去开发的，也就是Producer本地会管理TCP连接通道，Producer在创建的时候会通过bootstrap.servers配置创建连接，只要连上一个Broker就能读取到Broker集群，也就能与所有的Broker创建TCP连接，创建连接之后如果有长时间空闲的连接Kafka会对连接进行强制关闭，但是Kafka的Producer会每5分钟（metadata.max.age.ms配置）去更新集群信息，强刷一次元数据信息保证元数据的一致性。

**何时创建TCP连接**
1. 创建Producer的时候会去创建TCP连接通道
2. 发送消息的时候
3. 更新元数据信息之后

**何时关闭TCP连接**
1. 主动关闭（比如进程关掉之类的）
2. 被动关闭 比如前面说的空闲连接强制关闭


### Kafka Consumer消费端

#### Kafka Consumer TCP连接管理
Kafka Consumer创建TCP连接有三个时机创建TCP连接：

**1.发起FindCoordinator请求时**

负责消费者组的组成员管理和各个消费者的位移提交管理，当消费者首次调用poll方法拉取消息时候就会发起一个FindCoordinator请求，希望Kafka集群告诉它哪个Broker是管理它的协调者，在这一步消费者会创建一个Socket连接。

**2.连接协调者时**

第一步知道集群中哪一个是协调者之后，会与该协调者进行Socket连接，只有成功连入协调者，协调者才能开启正常的组协调操作，比如加入组、等待组分配方案、心跳请求处理、位移获取、位移提交等，所以这一步也会创建Socket连接。

**3.消费数据时**

消费者消费数据是根据订阅的分区来的，消费者要为每个分区领导者副本所在的Broker创建Socket连接，假如消费者消费了A、B、C三个分区，这三个分区分别在B1、B2两台机器，那么消费者需要创建与B1、B2的Socket连接。


Kafka Consumer关闭TCP的时机：

**1.主动关闭**

消费者主动执行close方法，那么Consumer所有的连接都会关闭。

**2.主动关闭**

Consumer端有一个参数connect.max.idle.ms（最大空闲时间默认为9分钟），如果在这个时间段都没有任何请求，那么这个连接也会被关闭。

#### Kafka Consumer 重要的参数

**bootstrap.servers：集群地址，一般设置三个，不需要设置全部的集群地址**

**group.id：消费组id**

**fetch.min.bytes：消费者每次poll消息最小值，默认为1b，如果想要提高吞吐量则调大这个值，但是有一定的延迟性**

**fetch.max.bytes：消费者每次poll消息最大值，默认为50M**

**fetch.max.wait.ms：与参数fetch.min.bytes对应，因为不能因为获取不到fetch.min.bytes值的大小而一直阻塞客户端，必须设定一个合理的等待值，如果对延迟敏感的消息，这个值可以设置的稍微小一些**

**max.partition.fetch.bytes：消费端每个分区返回消费端最大数据量，默认值为1M**

**max.poll.records：消费端每次poll消息条数最大值，默认为500，一般不要设置特别大，因为如果下次消费还未消费完的话可能会导致消费端重平衡，导致消费阻塞**

**max.poll.interval.ms：这个表示两次调用poll方法的最大时间间隔，默认为5分钟，如果超过5分钟Kafka的协调者则会认为这个消费者挂掉，会进行提出组处理**

**connections.max.idle.ms：这个参数指定客户端每个连接最大空闲时间，默认为9分钟，超过这个时间客户端就会关闭这个连接**

**exclude.internal.topics：这个参数一般不用动，表示是否支持消费端用subscribe（Pattern）来订阅内部主题，默认值为true表示消费端只能通过subscribe（Collection）来订阅内部主题，设置为false则没有这个限制。PS：内部主题为__consumer_offsets位移主题、__transaction_state事务主题**

**receive.buffer.bytes：这个参数用来设置Socket接受消息缓冲区的大小，默认值为64KB**

**send.buffer.bytes：这个参数用来设置Socket发送消息缓冲区的大小，默认值为128KB**

#### Consumer位移提交

Kafka早期位移提交设计是放在Zookeeper中的，但是Zookeeper设计理念是基于CP的，所以不太适合高并发的写，而且放在Zookeeper中也有局限性，必须依赖Zookeeper，后面Kafka位移提交是采用了内部主题，Kafka有一个内部主题__consumer_offsets来储存位移，默认有50个分区，每个分区有三个副本。

**Kafka Consumer每次启动的时候，如果内部主题还未创建，则会默认创建__consumer_offsets分区数为50个**

**Kafka Consumer每条消息都对应一个offset值，位移提交就是将消费的位移值写入到内部主题中**

**Kafka有一个清除策略，会定期去清除位移主题中已经过期的消息来防止位移主题无限膨胀**

**Kafka位移提交分为手动提交和自动提交，自动提交比较省事但是容易丢消息，手动提交编码复杂一点但是数据安全**

#### Consumer重平衡

触发重平衡三个点：

**1.消费组内成员变动**</br>
**2.订阅主题数量发生变化，这个主要是针对那些通过subscribe（Pattern）来订阅主题的消费组**</br>
**3.订阅主题分区数发生改变**</br>

这里我们只针对第一个点进行讨论，第二个和第三个属于无法避免的，消费端尽量明确消费Topic，尽量少写正则来订阅主题数，然后尽量减少分区数的改动。

至于第一个点组内成员变动又分为以下几个情况：

**1.消费者消费时间过长，导致两次poll的时间超过了max.poll.interval.ms默认时间（5分钟），这种情况我们尽量把max.poll.interval.ms值设置大一些，或者是每次拉取的条数减少，又或者减少消费者的业务逻辑缩短消费时间**

**2.消费者心跳未能及时发送导致被踢出组，这种情况我们采用session.timeout.ms、heartbeat.interval.ms两个参数来调优，session.timeout.ms默认为10S，如果Kafka协调者在10S内未收到消费端的心跳包就踢出组，heartbeat.interval.ms为消费端发送心跳的时间间隔，我们可以把时间间隔设置的稍微小一点保证在session.timeout.ms内能发送3个心跳包来避免误杀的情况**





