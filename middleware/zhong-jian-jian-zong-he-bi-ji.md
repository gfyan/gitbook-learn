# 中间件综合笔记

### 目前市面上都有哪些中间件？

**缓存中间件**
1. Redis
2. Memcached

**消息中间件**
1. RabbitMQ（Scala语言写的）
2. RocketMQ（阿里消息中间件，Java写的）
3. ActiveMQ（老古董，apache出的）
4. Kafka（LinkedIn公司的）
5. MetaMQ
6. ZeroMQ

**搜索引擎中间件**
1. ElasticSearch

**分布式中间件**
1. Codis（代理中间件，管理Redis，做分片处理）

**限流中间件**
1. Sentinel

**数据库相关中间件**
1. TDDL（分库分表查询）
2. Sharding JDBC（分库分表查询）
3. Sharding Proxy（分库分表查询）
4. MyCAT （分库分表查询）
5. Seata（分布式事务框架）
6. Canal（基于binlog的数据同步中间件）
