# 面试相关


### 在项目中经常遇到定时触发逻辑，比如订单创建15分钟后还未支付需要对用户进行通知，又比如订单回调需要做限定次数且每次间隔时间呈递增式进行回调，对于这种任意超时时间的定时触发业务该怎么设计？

1. 单独设计一个微服务作为定时触发中心，该中心维护一个核心业务表，定时触发表。
2. 定时触发中心对业务系统暴露接口，定时任务创建、定时任务取消、定时任务查询，每个定时任务执行时都调用对应的业务方。
3. 所有的业务系统凡是需要进行定时触发，都请求该服务进行定时任务创建。
4. 定时任务中心创建一个RocketMQ的Topic做延时消息topic，
5. 定时触发中心收到定时任务时进行定时任务创建，定时触发中心服务启动一个分布式定时任务，每十分钟对定时任务表进行扫描，扫描最近十分钟需要触发的任务，扫描到数据后将数据发放至阿里云RocketMQ的延时消息中，延时时间为定时触发时间-当前时间，然后消息投递成功后对定时任务状态进行更改，改为定时任务已投递。
6. 定时触发中心创建对应的一个group ID进行消费对应Topic中的消息，最终在Listener中实现最终的触发业务逻辑。

**如果需要做定时任务取消或者删除操作，直接将定时任务中心的定时任务表状态置为已取消即可，即便是已投递的消息，在Listener中进行状态判断即可，如果是已取消 或者消费的时间已超过精度值则直接忽略这一条定时触发任务。**

以上便是一个简易的定时任务触发中心。


### 你说你对订单创建接口进行了优化，而且最终将TPS提升了80%，将订单错误率降低到了1%，请说说具体是怎么设计的？

这一块之前主要是观察到了订单创建接口在某些业务高峰期会出现慢接口日志，所以我就对这一块的业务进行了梳理工作，然后对慢接口日志进行分析，慢接口主要出现的特点是公司平台的订单创建业务会出现慢接口，因为平台这边有一个新人注册即送红包功能，新人注册创建订单之前的业务是创建订单并且需要对出账信息和入账信息进行校验，这一块是通过cloud接口去调用的，然后接口调用的同时会如果用户钱包为注册的话需要同步进行注册操作，这是一个致命错误，主要慢就是在这里，而且平台这边出账之前只有一个账号，所以导致了只要是新用户红包订单过来，特别是并发量大的情况一定会阻塞后续的平台订单创建流程。

找到原因之后我们就对这一块的业务进行了整合优化。

1. 首先平台这边出账我们采用了一个单独的账号，对平台出账业务进行细粒度剥离，防止业务之间的影响，因为出账都是一个账户的话，多业务只能串行执行。
2. 将之前的钱包创建流程业务提前，用户注册之后用户中心就将注册消息发送至青团宝钱包管理服务这边来进行钱包注册一切逻辑，然后红包订单创建不对入账钱包进行校验，这一步逻辑放在了异步流程里面，订单创建成功，商户出账账号扣款成功之后就往青团宝用户钱包服务中心发送一条异步消息，用户钱包中心消费入账消息进行钱包入账操作。
3. 将扣款流程放在最后面，因为出账账号往往会出现并发操作，刚开始扣款流程放在最前面的，但是这么写会导致行锁的时间比较长，改造之后发现并发的情况下时间上会有部分缩短。
4. 代码层级上的优化

### 你之前将订单数据查询接口进行了优化，通过redis做了缓存，并且获得了qps翻倍的增长，说说具体是怎么实现的？

我先说业务背景，这个业务功能是针对于商户pc端一个订单统计页面，这个数据页要求展示当天的订单完成量以及订单完成金额，最早的时候是直接统计订单表的，但是后面随着业务量增大，特别是平台商户，每次打开这个页面的时候都会出现卡顿的情况，所以针对这一块我们做了优化，单独做一个数据统计表，以天为单位进行储存统计数据，然后同时将当天的数据存入redis中，每次商户打开这个统计数据界面，前端接口过来之后，后端拿取数据都会先去redis中看有没有缓存的数据，如果没有去缓存表中查询，如果都没有就直接统计订单表，然后订单明细数据进行分页展示。


缓存数据具体设计是通过mq去实现的，每次订单完成后，订单数据都会异步发送至订单对应的Topic中，针对缓存我们单独建了一个缓存处理的Group对其Topic中的消息进行消费，消费的时候将订单的数据进行整合至订单数据缓存表中，然后将其数据设置到redis缓存中。

**面试官可能会问的问题：**

1.如果缓存表中没有数据，redis中也没有，你现在的逻辑是直接去订单数据表中查询，这样设计有什么问题。
	答：存在问题，比如mq消息消费延迟，导致缓存表中不存在数据，然后前端进来特别多的数据请求，这个时候DB压力会很大，严重的会直接影响到其他的业务，比如订单的提交都会变慢，这种情况出现我觉得可以做一个缺省页出来，对数据进行兜底，即便是缓存表中的数据没有，那么也不要直接去订单表查询，直接返回前端一个特定的code，前端展示数据统计中，请稍后再试，因为数据统计业务并非很急切对于商家来说数据统计页看不到和下单失败影响点完全不在一个level上。


### 你说你改造了三方验证服务，实现多线程并发银行卡校验、实名信息校验功能，最终也是实现了性能上很大的优化，说一下具体的实现？


嗯，还是先说下业务背景，三方验证服务承接平台这边所有的实名认证服务以及银行卡校验服务，通过cloud接口暴露给内部业务，使用三方验证服务一般都有商户的实名认证、用户的实名认证、商户打款直接到卡功能、用户绑定银行卡等功能。

这一块主要优化就是使用了OkHttp去请求三方验证接口，最早的是采用的是HttpURLConnection，一方面是自带的HttpURLConnection编码就比较复杂，而且性能也不是很好，后面改用了okHttp，因为okHttp本身自带一个连接缓冲池，默认是5个连接通道，存活时间是5分钟，我们将通道连接池提升到50，其性能得到了很好的提升。



### 你说你与团队一起开发了千人千面，这个千人千面你主要参与了哪些工作，具体怎么设计的？

参与工作主要是参与前期业务架构设计、将报名单服务的报名业务数据实时输送到大数据中心，然后针对用户兼职打标这一块是我做的。

千人千面是我们平台对用户首页兼职的本地精选做的一个优化功能，之前的本地精选兼职列表获取业务逻辑很简单，就是直接通过用户当前的性别以及地区进行兼职筛选，没有针对不同的用户进行打标功能，而且点击/报名比率也一直很低，所以我们就讨论将这个本地精选进行优化。

主要设计是这样的：

原逻辑（搜索）：

    根据城市ID+过滤排队兼职+过滤线上兼职+过滤白名单公司，取50个兼职，随机24个展示。

新逻辑（千人千面1.0）：

     数据层面：

          1.离线数据：当天0点开始离线计算，根据三个月活跃用户和他们所在的常登陆地区计算每个用户的200个兼职池子，计算逻辑见算法部分；并计算一份每个城市的默认推荐池子供新用户使用；

          2.前台逻辑：根据用户和城市ID随机获取24条数据。
     位置变更:
          先根据用户ID拿数据，如果和常驻城市不匹配，拿当前城市的默认数据。

计算算法是：

1、用户：最近三个月活跃用户(登录使用)
2、兼职：截止日期大于当天的所有线下兼职
3、获取用户近期常登录城市，并匹配到该城市下的所有线下兼职
4、针对每个用户的匹配兼职，进行一系列规则过滤操作：

    性别：如果兼职所需性别与用户性别一致，10分；

               如果两者性别一致都是未知0，5分；

               如果两者性别不一致且有一方是性别未知0，3分；

               性别不一致，0分； 

    学历： 如果用户学历大于等于兼职要求学历，10分；如果兼职要求学历通用，5分；

                如果两者学历要求通用，3分；

                不匹配，0分；

    类目： 用户报名过同类目，10分；

                用户浏览或者收藏过的类目，5分；

                其他，3分；

5、过滤掉完全不符合的兼职岗位，计算好各项规则分数后，加入各项权重值(类目0.5，性别和学历各0.25)

6、得到最终用户和对应兼职的匹配分数，加入随机打乱的系数，这样得到每个用户唯一的排序后兼职列表给后端；

默认城市推荐：

1、先取出每个城市下面所有的过期时间大于当天的线下兼职；
2、分别计算每个城市下的兼职的剩余可报名人数；
3、分别计算每个城市下的兼职的之前用户行为，兼职的曝光点击比；
4、综合可剩余报名人数和兼职的曝光点击比来进行兼职热门排序，
      其中头部热门城市的兼职数比较大，排序后取前面300个兼职作为该城市的兼职推荐池子，
      其他小于200个兼职的城市只是按照排序后全部作为推荐池子；









