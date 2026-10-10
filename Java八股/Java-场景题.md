# Java场景题



1.用过线程池吗?一般用来处理哪些任务?
2.线程池满了怎么办?除了拒绝任务还有哪些处理方式?
3.你怎么理解程序崩溃?编译错误和运行时崩溃有什么区别?
4.程序运行过程中异常退出，应该怎么排查?
5.Java 程序异常退出有哪些情况?会留下什么日志?怎么定位到具体线程和代码位置?
6.进程 CPU 占用很高，可能是什么原因?怎么排查?
7.任务堆积一定会导致 CPU 占用升高吗?为什么?
8.如果任务处理不过来，为什么不直接把线程池开到 100、200 个线程?
9.线程上下文切换为什么有开销?具体需要做哪些事情?
10.用户态切换到内核态为什么有开销?
11.为什么要区分用户态和内核态?



12.一次请求通过 RPC 框架发送到对端，整个过程是什么?
13.RPC 中的代理拦截是什么意思?具体拦截了什么?
14.请求从应用层进入网络栈，到发送出去，还会经过哪些处理?
15.请求数据很大时，传输层和网络层会怎么处理?会把整个请求一次性发出去吗?
16.数据被拆分后，接收端怎么恢复出完整数据?
17.如果中间丢了一两段数据，是不是要把整个大文件重新传一遍?
18.对端进程退出、机器突然断电、网络不通，另一端分别能不能感知?
19.TCP 的 RST 是什么意思?应用程序会感知到什么?
20.如果对端挂了，另一端会一直等待吗?怎么避免?
21.怎样主动检测对端是否可用?心跳机制怎么实现?



22.给你一大块内存，让你实现内存申请和释放，怎么设计?
23.申请内存时，怎么找到“合适”的空闲块?
24.释放内存后，空闲块什么时候合并?具体怎么合并?
25.如果 4 KB 和1MB 的内存申请交错进行，释放小块后仍然无法满足大块申请，怎么减少内存碎片、提高利用率?



26.JVM 怎么管理内存?一个新对象的内存具体怎么分配?
27.新生代和老年代怎么工作?Eden空间不足时怎么处理?
28.客户端内存很小，文件又非常大无法一次性读入内存，怎么上传到服务端?
29.Redis bitmap 最多支持多少位?可以用于哪些场景?
30.进程间通信有哪些方式?
31.线程之间有哪些通信和同步方式?
32.手写一个线程安全、有容量上限的生产者消费者队列，提供 push和 pop 方法。
33.手撕算法:对一个乱序单链表进行原地排序。







## Java-Collection相关



### Q1：大量数据put进hashmap

如果有很大的数据量需要put进hashmap中，应该怎么做

A1:

1. 容量预分配，避免扩容风暴
2. 谨慎调整负载因子
3. 重写hashcode函数，避免产生大量冲突
4. 选择合适的第三方map实现
5. 分治策略，水平分片，把数据分散到多个hashmap实例中



## MySQL相关

### Q1：分表策略

如果想定下有千万级的数据，每次查询会很耗时，该怎么办？答：先考虑分表，按想定类别进行区分。追问：如果各个想定下的数据量不均匀，有的数据量很大，有的很小，分表的意义在哪里，应该怎么优化

**A1：**

三个方案：

1. **物理打散**：用 ID哈希取模做分库分表，保证数据物理均匀，消除单表瓶颈。
2. **缓存挡击**：引入 Caffeine 本地缓存 + Redis 分布式缓存，缓存命中率高的热点数据，让大部分请求根本不落到DB。
3. **冷热分离**：针对极少数数据量特别大的‘超级类别’，按时间或热度拆成热表和冷表，查热表为主，异步查冷表为辅。



场景题：热点榜单实时更新如何保证高并发？数据结构如何设计？数据库如何选型？



## 系统设计相关

### Q1：短链系统设计

>设计一个QPS1000的短链系统，要求短链唯一，跳转时间小于50ms，你会怎么设计这个读写链路

#### 整体架构

第一步：明确核心矛盾（破题）

面试官想听的是你对**读写分离**和**最终一致性**的理解。

- **读（跳转）**：要求极低延迟（<50ms），QPS 1000。**策略：全缓存，尽量规避DB查询。**
- **写（生成）**：要求全局唯一，高可用。**策略：利用发号器预生成ID，异步落库。**

------

第二步：读链路设计（跳转重定向）—— 核心是“多级缓存”

用户访问 `short.com/abc123` 到返回 `302` 状态码，必须控制在 50ms 内。

**1. 浏览器/客户端侧（可选）：**

- 利用浏览器本地缓存（LocalStorage）或 HTTP 缓存头（Cache-Control），对热点短链做极短期缓存（如 1分钟），减少后端压力。

**2. 服务端读链路（重点）：**

- **L1 本地缓存（Caffeine/Guava Cache）**：挂在应用进程内。使用 LRU 算法，存储最热门的 Top 20% 短链。命中则直接返回（耗时 < 1ms）。
- **L2 分布式缓存（Redis Cluster）**：如果本地缓存未命中，查询 Redis。**数据结构建议用 Hash**，存储 `{key: long_url, expire_time}`。Redis 单节点 QPS 可达 10w+，耗时约 0.5ms - 2ms。
- **L3 数据库（DB）兜底**：如果 Redis 因宕机或缓存穿透未命中，查询 MySQL（或 TiDB）。**必须加布隆过滤器**，防止恶意请求直接打穿 DB。

**3. 异常处理与性能保障：**

- 如果 DB 查询发现该短链已过期或不存在，直接返回 404，并**写入一个空值占位符到 Redis（过期时间极短，如 1分钟）**，防止缓存穿透。
- **异步上报**：每一次跳转请求，通过 MQ（如 Kafka）异步记录访问日志（用户IP、设备等），**绝对不阻塞主线程**，确保主链路纯粹只做 KV 查询。

> **读链路耗时估算**：L1命中（1ms） -> L2命中（2ms） -> L3兜底（10ms，加上布隆过滤器校验）。均满足 < 50ms。

------

第三步：写链路设计（生成短链）—— 核心是“去中心化发号器”

要求短链唯一，且不能依赖数据库自增（容易成为瓶颈且重号）。这里推荐 **Snowflake（雪花算法）变种**。

**1. 生成唯一短码（不查重）：**

- 采用 64位 Snowflake ID（1位符号 + 41位时间戳 + 10位机器ID + 12位序列号）。
- 将这个 10进制 Long 型 ID，**转换为 62进制（a-zA-Z0-9）** 字符串，长度仅 6~7 位（如 `aB3XkQ`）。
- **关键点**：既然 Snowflake 保证全局唯一，那么转换后的短码也必然全局唯一，**省去了查重步骤**，极大提升了写入速度。

**2. 读写链路数据同步：**

- **同步写入 DB（强一致性）**：将 `{short_code, long_url, create_time}` 写入 MySQL。这一步是唯一较慢的步骤（约 5ms-10ms），但 QPS 1000 对于 MySQL 写入是完全可承受的。
- **异步更新缓存（Cache-Aside 策略）**：写入 DB 成功后，**立即将数据写入 Redis**（设置合理的 TTL，如 24小时），并**失效（或更新）本地缓存**。

**3. 防止重复长链生成相同的短链（去重优化）：**

- 面试官可能追问：“如果用户用同一个长链生成两次，你给两个不同短链吗？”
- 推荐做法：**允许生成两个不同短链**（因为加了时间戳），但为了节省存储，可在业务层增加一个 **布隆过滤器 + 长链MD5索引**。如果用户请求生成，先查长链 MD5 是否在 Redis 中存在，若存在直接返回旧短链；若不存在则走 Snowflake 生成新码。

------

第四步：应对 QPS 1000 的具体落地配置

- **应用层**：横向部署 **3~5 台** 4核8G 机器（Nginx 负载均衡），单机处理 300 QPS 毫无压力。
- **Redis**：部署 **主从 + 哨兵** 或 **Cluster 集群**，内存控制在 10G 以内（存储热点短链）。
- **DB**：MySQL 分库分表（按短码 Hash 分 16 张表），写入 QPS 1000 完全够用，不需要引入太复杂的中间件。

------

第五步：50ms 跳转时间的终极保障（熔断与降级）

- **设置超时阈值**：在 RPC 或 HTTP Client 层设置 `connectTimeout=500ms`，`readTimeout=30ms`。如果 Redis 响应超过 30ms，直接**熔断**，降级查询本地缓存，若本地也没有，直接返回兜底页面（如官网首页），保证用户体验。
- **CDN 预热**：如果是静态化的活动页短链，可将 302 跳转前置到 CDN 边缘节点，响应速度可达 10ms 以内。



#### 存储层设计

“针对这个系统的存储层，采用**冷热分层**的设计：

- **热数据**放在**本地缓存（Caffeine）**，扛住 80% 的读流量，响应 < 1ms；
- **温数据**放在**Redis Cluster**，使用 **Hash 结构**存储元数据，响应 < 2ms；
- **冷数据**持久化在**MySQL**，按 `short_code` Hash 分 16 张表，配合唯一索引做最后的防重兜底。

写入时采用**先 DB 后 Cache** 的策略，保证数据强一致性；更新时用**延迟双删**防止脏读。整个存储层的设计核心是**让 99% 的请求不查库**，从而用最少的成本满足 50ms 的延迟要求。

如果未来 QPS 增长到 1万+，我会把 MySQL 替换为 **TiDB** 或 **Cassandra**，并用 **Pulsar** 做缓存更新的异步最终一致性，但目前 QPS 1000 的规模，这套分层存储架构是性价比最高的方案。”



#### Redis发号设计

"针对短链生成的高并发重复问题，我采用 **Snowflake 本地发号** 作为核心方案：

**第一层保障**：通过 `synchronized` 保证同一机器内线程安全，通过 **41位时间戳 + 10位机器ID + 12位序列号** 的组合保证全局唯一，其中序列号在同一毫秒内可生成 4096 个ID，完全覆盖 QPS 1000 的需求。

**第二层保障**：机器ID通过 **Redis 或 Zookeeper** 自动分配，应用启动时注册，下线时自动释放，避免不同机器ID冲突。

**第三层保障（兜底）**：如果发生时钟回拨，我会让系统**等待时钟追上**，或者直接切换到 **Redis 批量自增** 模式（每次预取 1000 个ID），作为降级方案，保证发号服务始终可用。

**最关键的一点**：生成ID后，我们直接将其转换为 62 进制短码，**无需查重直接写入数据库**，利用数据库的唯一索引作为最后的防重保障。如果发生极低概率的ID碰撞（概率 < 1/10^18），MySQL 会抛出 `Duplicate Key` 异常，应用层捕获后重试一次即可。这样既保证了唯一性，又省去了每次生成时查库的网络开销。"



### Q2：秒杀系统设计

> 设计秒杀系统，1万人抢100件商品吗？然后每个人限购一件，从前端到后端链路会怎么做

```
┌─────────┐
│  用户端  │ (浏览器/APP)
└────┬────┘
     │ ① 页面静态化 + CDN
┌────▼─────────────────────────────────────────────┐
│  Nginx + CDN (静态资源)                          │
└────┬─────────────────────────────────────────────┘
     │ ② 限流 + 动静分离
┌────▼─────────────────────────────────────────────┐
│  Gateway / 负载均衡层                            │
│  - 令牌桶限流 (整体 QPS 控制)                    │
│  - 黑名单/IP 限流                                │
└────┬─────────────────────────────────────────────┘
     │ ③ 业务校验
┌────▼─────────────────────────────────────────────┐
│  秒杀服务集群 (无状态水平扩展)                   │
│  - 参数校验                                      │
│  - 用户资格校验                                   │
│  - 防重复下单 (Redis SETNX)                     │
└────┬─────────────────────────────────────────────┘
     │ ④ 库存扣减 (核心)
┌────▼─────────────────────────────────────────────┐
│  Redis (库存扣减 + 限购记录)                     │
│  - Lua 脚本原子扣减                              │
│  - 记录已购用户                                  │
└────┬─────────────────────────────────────────────┘
     │ ⑤ 异步落库 增加本地消息表，增加mq可靠性
┌────▼─────────────────────────────────────────────┐
│  MQ (RabbitMQ/Kafka)                            │
│  - 削峰填谷                                      │
│  - 异步创建订单                                  │
└────┬─────────────────────────────────────────────┘
     │ ⑥ 最终一致性
┌────▼─────────────────────────────────────────────┐
│  数据库 (MySQL)                                  │
│  - 订单表 + 库存表 (乐观锁兜底)                  │
└──────────────────────────────────────────────────┘
```

#### Lua脚本相关

> 秒杀时Redis里既要判断库存又要判断一人一单，为什么要放进Lua，而不是拆成多个Redis命令？ 

Redis 执行 Lua 脚本时：**整个脚本期间 Redis 不会执行其他命令。**

Redis 是单线程执行命令：

```
请求A:

Lua脚本
|
|-- 判断库存
|
|-- 判断用户
|
|-- 扣库存
|
|-- 写购买记录
|
结束


请求B:

等待
```

所以：

```
库存判断
+
用户判断
+
库存扣减
+
用户记录
```

#### Redis与MySQL最终一致

> Redis已经预扣库存，但后续MQ发送失败或数据库下单失败，Redis和MySQL怎么保证最终一致？ 

**秒杀系统通常不追求强一致，而追求最终一致。**

情况1：Redis扣成功，MQ发送失败

情况2：MQ成功，但是消费者失败

情况3：数据库写成功，但是MQ ACK失败

方案一：MQ可靠投递 + 重试（解决消息丢失）

生产端确认机制

例如RabbitMQ：

```
Seckill Service

    |
    | publish
    ↓

RabbitMQ

    |
    | confirm
    ↓

生产者收到ACK
```

如果：发送失败，则：retry

方案二：Redis预扣 + 本地事务日志

Redis扣库存的时候：

同时记录：

```
seckill_order:
{
 userId,
 productId,
 status=INIT
}
```

例如：

Redis：

```
stock=99

order:
10001:user888
status:init
```

然后：后台补偿任务：扫描：status=INIT；发现：超过30秒重新发送MQ。

方案三：MQ消息最终一致性（主流）

实际互联网系统更常见：

引入：订单状态机

例如：订单表：

| id    | user | 商品 | 状态    |
| ----- | ---- | ---- | ------- |
| 10001 | 888  | 手机 | INIT    |
| 10002 | 999  | 手机 | SUCCESS |

流程：

```
Redis扣库存

↓

MQ

↓

创建订单

状态 INIT

↓

支付成功

↓

SUCCESS
```

**Redis负责“秒杀阶段的高并发控制和临时状态管理”，MySQL负责“订单业务的最终事实记录和生命周期管理”。两者不是简单的数据复制，而是职责互补。**



> 如果订单已经写入MySQL，但MQ消息ACK失败怎么办？

```
MQ

 ↓

订单服务消费消息

 ↓

MySQL写订单成功

 ↓

消费者返回ACK给MQ

 ↓

ACK失败
```

**数据库唯一约束（最常用）**

例如订单表：

```
CREATE TABLE orders(
 id bigint,
 user_id bigint,
 product_id bigint,

 UNIQUE(user_id, product_id)
)
```

第一次：INSERT成功

第二次：Duplicate Key

订单服务捕获：已经创建过、直接返回成功

**保证订单幂等性，防止重复消费**

如果MQ在redis扣减库存之前，那么重复消费的问题，redis库存也需要做幂等性判断，复杂度提升

因此，秒杀资格校验和库存扣减放在MQ之前，订单消费阶段只负责落库。

#### 本地缓存Caffine

> Caffeine + Redis二级缓存具体缓存什么？活动开始前怎么做缓存预热？ 

在秒杀系统中可以采用 Caffeine + Redis 二级缓存架构。Caffeine作为JVM本地缓存，主要缓存活动配置、商品详情、秒杀规则等读多写少的数据，减少应用访问Redis的网络开销；Redis作为分布式缓存，负责多实例共享数据。但库存、用户购买资格等强一致数据不放入Caffeine，而由Redis Lua统一控制。活动开始前，通过定时任务提前加载活动信息到Redis，并通过消息通知各应用实例完成Caffeine预热，避免秒杀开始瞬间大量请求穿透到数据库。



#### 如何处理缓存的数据特别大的情况

要先区分两种“大”：**单条缓存值很大**，还是**需要缓存的记录总量很大**。处理方式不同。

如果是单条值很大，例如把一家店的所有优惠券、评论和图片信息塞进一个 Redis key，建议拆开：店铺基本信息单独缓存，优惠券和评论按页缓存，图片只存 URL、不存图片内容。接口也按需取数据，避免一次读取和传输整个对象。压缩可以作为补充，但会增加 CPU 开销，应先测量再决定。

如果是记录总量很大，不必把数据库全部搬进 Redis。优先缓存访问频繁的数据，设置合理 TTL 和内存淘汰策略；低频数据直接查数据库。还要监控 Redis 内存、命中率和大 key，避免缓存反过来成为瓶颈。

#### 创建订单耗时很长怎么办

HTTP 接口做 Redis 预扣库存并发送 Kafka 消息，然后消费者异步写数据库。因此，接口返回快，不等于订单已经创建完成。

建议先分别测这两个时间：

- **接口响应慢**：记录 token 校验、Redis 生成 ID、Lua 执行、Kafka `send` 各段耗时，重点看 Redis 连接池等待、Kafka 连接与应用日志开销。
- **接口已返回订单 ID，但数据库很久才有订单**：重点看 Kafka consumer lag，以及查重、扣库存、插入订单各段耗时。当前订单落库逻辑 (line 105)每单都要获取 Redisson 锁，再进行数据库查重、扣库存和插入；监听器没有显式配置消费并发。

查明瓶颈后再处理：数据库查重慢就确认 `(user_id, voucher_id)` 联合索引**已在实际数据库执行**；Kafka 积压就评估增加分区和消费者并发，但同一张券的库存行仍可能成为数据库锁热点，不能只靠加线程解决。



热点数据怎么识别？如果某个服务实例重启，刚好错过MQ广播的缓存失效消息怎么办？



#### 单实例系统改为分布式

> 单实例秒杀改造成分布式后，首先要消除所有单机状态。比如本地锁要改成Redis Lua等分布式原子操作，本地库存和一人一单记录要放到Redis，本地Session改成JWT或Redis共享Session，订单ID改成分布式ID生成，服务本身设计成无状态，方便水平扩容。
>
> 在整体架构上，我会采用Gateway + 秒杀服务集群 + Redis + MQ + MySQL。入口通过限流、防刷、验证码等尽可能过滤无效请求；Redis通过Lua原子完成库存判断、重复购买校验、库存扣减和购买资格记录；成功请求进入MQ削峰，消费者异步创建订单和更新数据库。
>
> 稳定性方面主要通过入口限流、服务水平扩展、Redis高可用、MQ削峰、熔断降级来保护系统，尤其Redis故障时不会把请求直接降级到MySQL，而是优先快速失败，防止数据库雪崩。
>
> 一致性方面，Redis负责实时库存和购买资格，MySQL作为最终事实来源。Redis内部通过Lua保证原子性，数据库通过`stock > 0`条件更新防超卖，通过`user_id + voucher_id`唯一索引保证一人一单。Redis、MQ、MySQL之间采用可靠消息、消费幂等、失败重试以及对账补偿保证最终一致性。

最后记一个很适合面试的总结：

**单机转分布式，本质上是四件事：状态共享、并发协调、流量治理、最终一致性。**





### Q3：积分排行榜

> 设计一个千万级用户的每日更新积分排行榜，要求支持高效的排名查询和积分更新，怎样设计

#### 整体架构

```
              用户请求
                 |
              API Server
                 |
        +----------------+
        |                |
   积分更新接口      排名查询接口
        |                |
        |                |
       MQ              Redis
        |
        |
   积分计算服务
        |
        |
    MySQL/DB
```

核心：

- **Redis维护实时排行榜**
- **MySQL保存最终积分数据**
- **MQ削峰异步更新**

#### 积分更新流程

```
用户行为
   |
   |
业务服务
   |
   |
发送积分事件
   |
   |
MQ(Kafka)
   |
   |
积分服务消费
   |
   |
Redis ZINCRBY
   |
   |
MySQL异步落库
```



#### 查询排名

```shell
ZREVRANGE daily_rank 0 99 WITHSCORES
```

#### 每日排行榜

因为是每日更新，所以不能一直使用同一个key。

设计：

```
daily_rank:20260902
```

每天一个排行榜。

凌晨：

生成新榜：

```
daily_rank:20260903
```

旧榜：

```
daily_rank:20260902
```

保留一段时间。

#### 千万用户初始化

MySQL批量导入Redis

#### 积分一致性

**方案1：MQ保证最终一致**

```
积分事件
 |
Kafka
 |
消费者
 |
Redis
 |
DB
```

Kafka消息可重试。

失败：重新消费。

**方案2：事件表**

MySQL：

```
score_event

id
user_id
change_score
status
```

先记录：

```
增加20分事件
```

然后异步执行。

保证：

- 可重放
- 可追踪

#### 总结

对于千万级用户每日积分排行榜，我会采用 Redis Sorted Set + MySQL 的架构。Redis负责实时排名查询，MySQL负责积分持久化。用户积分变化后，通过MQ异步进入积分服务，积分服务使用ZINCRBY更新Redis排行榜，同时异步落库MySQL保证最终一致。

排行榜按照日期设计key，例如daily_rank:20260902，每天生成新的排行榜。Top N通过ZREVRANGE查询，用户排名通过ZREVRANK查询，附近排名通过先查rank再查询范围实现。

对于千万级数据初始化，通过离线任务批量加载Redis，而不是实时扫描数据库。高并发场景下，通过MQ削峰、Redis Pipeline批量更新、Redis Cluster分片提升吞吐。

整体设计保证了积分更新O(logN)，排名查询O(logN)，能够支持千万级用户规模。

#### 追问

> 在千万级日活的排行榜场景中，选分布式锁、redis单命令原子操作、lua脚本这几种方案来处理积分更新，各自的优劣是什么，你最后该如何选择

在千万级DAU排行榜场景下，我不会优先使用分布式锁，因为积分更新是高频操作，锁竞争会明显降低吞吐，尤其热点用户会产生严重瓶颈。

如果只是排行榜积分累加，例如用户增加10分，我会直接使用Redis ZINCRBY这种单命令原子操作，性能最高，并且Redis本身保证并发安全。

如果积分更新涉及多个Redis数据结构，例如同时更新排行榜、用户等级、任务进度，我会使用Lua脚本，把多个Redis操作封装成一个原子操作。

如果涉及Redis之外的系统，比如数据库、奖励系统、消息通知，则不会使用锁或Lua，而采用MQ异步化，通过事件驱动保证最终一致性。

所以整体选择原则是：

**单Redis操作 → Redis原子命令；
 多Redis操作 → Lua脚本；
 跨系统事务 → MQ+最终一致性；
 分布式锁只用于复杂临界区且低频场景。**

### Q4：公司抽奖系统

> 设计公司几千人同时抽奖抽嘉年华门票系统，门票共500份，用户30%概率中奖、70%不中奖；
>
> 思考两点：①表结构如何设计；②核心抽奖接口怎么做，如何处理高并发、防止库存超卖。 



#### 表结构设计

1. 活动表 lottery_activity，记录一次抽奖活动。

```
lottery_activity
---------------------

id              bigint PK

activity_name   varchar

start_time      datetime

end_time        datetime

status          tinyint

created_time    datetime
```

例如：

| id   | name           | status |
| ---- | -------------- | ------ |
| 1    | 2026嘉年华抽奖 | 进行中 |

2. 奖品表 lottery_prize，定义奖品。

```
lottery_prize
---------------------

id              bigint PK

activity_id     bigint

prize_name      varchar

prize_type      tinyint

total_stock     int

remain_stock    int

probability     int

created_time    datetime
```

数据：

| 奖品       | 库存   | 概率 |
| ---------- | ------ | ---- |
| 嘉年华门票 | 500    | 30   |
| 谢谢参与   | 999999 | 70   |

这里概率不要存：

```
0.3
```

建议存整数：

例如：

```
3000
7000
```

总和：

```
10000
```

方便随机。

3. 用户抽奖记录表 lottery_record

这是最核心的表。

```
lottery_record
--------------------

id              bigint PK

activity_id     bigint

user_id         bigint

prize_id        bigint

result          tinyint

created_time    datetime
```

例如：中奖：

```
user_id=10001

prize_id=1

result=1
```

未中奖：

```
prize_id=null

result=0
```

关键索引：防止一个用户重复抽：

```
unique(activity_id,user_id)
```

4. 库存流水表 stock_record（推荐）

记录库存变化。

```
stock_record
------------------

id

prize_id

change_num

before_stock

after_stock

biz_id

created_time
```

例如：

```
门票

500 -> 499
```

方便：

- 审计
- 对账
- 排查超卖

> Redis库存和数据库库存，以哪一个为准，为什么？ 
>
> 用户点击抽奖按钮需要立刻前端展示抽奖结果，该如何设计？ 
>
> Redis扣减库存能否保证缓存库存一定准确？ 

在抽奖实时链路中，Redis库存作为**实时库存准入依据**；数据库库存作为**最终事实来源（source of truth）**。

用户点击抽奖后，需要同步返回结果，所以核心链路只完成必要操作：校验用户、概率计算、Redis Lua原子扣库存，然后立即返回中奖结果。数据库落库通过MQ异步完成。

Redis扣库存不能保证缓存库存和数据库永远一致，它只能保证Redis内部操作原子性。实际系统需要通过MQ可靠投递、消费幂等、失败重试、库存对账和补偿机制保证最终一致。



> 操作库存时，Redis查不到库存key（丢失/过期），有什么降级方案？ 



方案1：库存预热 + 永不过期（首选）

方案2：Redis Key不存在时，互斥恢复

```
请求
 |
Redis查询库存
 |
发现key不存在
 |
获取恢复锁
 |
查询DB库存
 |
重新初始化Redis
 |
继续秒杀
```

例如：

```
SETNX lock:stock:init:voucherId
```

只有一个线程恢复。

方案3：Redis恢复失败，快速失败



Kafka消费相关：原有4台服务消费，消费速率慢，扩容到8台，对消费速率是否有帮助？ 

Kafka同一个消费组内，同一个分区可以被几个消费者消费？该设计有什么好处？ 



### Q5：分布式锁设计

#### 为什么需要分布式锁

分布式场景下，多实例部署

```
服务A JVM            服务B JVM

synchronized         synchronized
      ↓                    ↓
   同一业务资源         同一业务资源
```

两个 JVM 的锁互相不知道，因此失效。

所以需要：

```
服务A ─┐
服务B ─┼──> 公共协调组件
服务C ─┘
```

大家共同竞争同一把锁。

#### 合格的分布式锁

* 互斥性：同一时间只能一个客户端获得锁；
* 防死锁：如果拿锁的机器突然挂了，没来得及取消锁，不能让锁永远存在。必须有过期时间；
* 只能释放自己的锁：锁必须记录：**锁的 owner。**
* 加锁、解锁必须原子：
* 可重入性：同一线程再次获取同一把锁时，最好能够成功，而不是把自己锁死。

#### redis实现基础分布式锁

加锁：

```
SET lock:key requestId NX PX 30000
```

解锁：

```
Lua：
if value == requestId
    DEL
```

解决了：

```
互斥性
+
死锁
+
误删别人的锁
+
原子释放
```

#### 过期时间如何确定

思路：

```
获得锁

TTL = 30s

后台线程定期续期
```

例如：

```
0s    获取锁，TTL=30s
10s   续期 → 30s
20s   续期 → 30s
30s   续期 → 30s
...
```

只要业务线程还活着，就不断延长锁。

业务完成：

```
unlock
```

停止续期。

如果服务器挂了：

```
WatchDog也没了
```

30秒后锁自动释放。

所以：

> **TTL解决死锁，WatchDog解决业务执行时间不可预测的问题。**



#### 可重入如何设计

例如：

```
lock.lock();

methodB();
```

而：

```
methodB() {
    lock.lock();
}
```

如果只是：

```
SET NX
```

第二次一定失败。所以需要记录：

```
owner
+
重入次数
```

可以用 Redis Hash：

```
lock:order:1001

{
    owner_A : 2
}
```

第一次加锁：

```
不存在
↓
owner_A = 1
```

第二次同线程加锁：

```
发现owner还是自己
↓
owner_A = 2
```

第一次unlock：

```
2 → 1
```

第二次unlock：

```
1 → 0
↓
真正DEL
```

整个过程仍然应该通过 Lua 原子执行。

这就是：

> **Reentrant Distributed Lock。**

Redisson实际上帮我们把这些复杂逻辑封装好了。



#### 没有抢到锁

主要有三种策略。

方式1：立即失败

```
tryLock()
```

没拿到：

```
直接返回
```

适合：

- 防重复请求
- 定时任务抢占

方式2：自旋重试

例如：

```
拿锁失败
↓
sleep 50ms
↓
重新尝试
```

但一定：

```
有限次数 / 有超时时间
```

否则造成Redis巨大压力。

方式3：等待通知

更高级的实现不会一直轮询。例如 Redisson 可以结合：

```
Pub/Sub
```

锁释放时通知等待者。降低无效轮询。





## 算法相关

### Q1：排序top K

长度为 n 的无序数组，找最大的 k 个

常见做法有：

方法一：排序后取前 k 个

- 时间复杂度：`O(n log n)`
- 空间复杂度：`O(1)` 或 `O(n)`，取决于排序实现
- 优点：简单
- 缺点：当 `k << n` 时，没必要全排序

方法二：维护一个大小为 k 的最小堆

- 遍历数组，先用前 k 个元素建一个**最小堆**
- 对后面每个元素：
  - 如果它比堆顶大，就替换堆顶，并调整堆
  - 否则跳过
- 遍历完后，堆里就是最大的 k 个元素
- 时间复杂度：`O(n log k)`
- 空间复杂度：`O(k)`
- 适合：`k` 远小于 `n`，且数据可以流式处理

方法三：快速选择（QuickSelect）

- 类似快速排序的分区思想
- 平均时间复杂度：`O(n)`
- 最坏时间复杂度：`O(n^2)`
- 空间复杂度：`O(1)` 或 `O(log n)`
- 缺点：需要能随机访问整个数组，通常要求数据都在内存里

**内存只能容纳约 1000 个数，数据总量 10 万，要找最大 1 万**

方案 A：分块排序，再归并取前 1 万

1. 把 10 万个数分成若干块，每块大小不超过内存限制，比如每块 1000 个。
2. 每块读入内存，排序后写回磁盘。
3. 得到 100 个有序块。
4. 对这些有序块做**多路归并**，从大到小取前 1 万个。
5. 归并时只需要每块读一个当前最大元素进内存，用最大堆维护这些块头。
6. 取满 1 万个就停止。

- 时间复杂度：分块排序 `O(n log b)`，归并取前 k 个 `O(k log m)`，其中 `b` 是块大小，`m` 是块数。
- 空间复杂度：`O(b + m)`，可以控制在内存限制内。
- 结果可以边归并边写到外部存储。

方案 B：如果允许“结果也写磁盘”，用最小堆 + 外部存储

如果内存能放下 1 万个结果，那最简单：

1. 维护一个大小为 1 万的最小堆。
2. 流式读取 10 万个数。
3. 比堆顶大就替换。
4. 最后堆里就是答案。



### Q2：LRU变种

**基础实现**

```python
class Node:
    def __init__(self, key=0, value=0):
        self.prev = None
        self.next = None
        self.key = key
        self.value = value


class LRUCache:

    def __init__(self, capacity: int):
        self.dummy = Node(value=-1)
        self.dummy.prev = self.dummy
        self.dummy.next = self.dummy
        self.capacity = capacity
        self.node_map = {}

    def get_node(self, key):
        if key not in self.node_map:
            return
        node = self.node_map[key]
        self.remove(node)
        self.put_front(node)
        return node

    def put_front(self, node):
        self.dummy.next.prev = node
        node.next = self.dummy.next
        self.dummy.next = node
        node.prev = self.dummy

    def remove(self, node):
        node.prev.next = node.next
        node.next.prev = node.prev

    def get(self, key: int) -> int:
        node = self.get_node(key)
        return node.value if node else -1

    def put(self, key: int, value: int) -> None:
        node = self.get_node(key)
        if node:
            node.value = value
            return
        node = Node(key, value)
        self.node_map[key] = node
        self.put_front(node)
        if len(self.node_map) > self.capacity:
            last_node = self.dummy.prev
            self.remove(last_node)
            del self.node_map[last_node.key]


# Your LRUCache object will be instantiated and called as such:
# obj = LRUCache(capacity)
# param_1 = obj.get(key)
# obj.put(key,value)

```

**并发控制版**

保留原来的哈希表 + 双向循环链表结构，只增加一把互斥锁，并且让 `get()` / `put()` 整个复合操作都在锁内执行。

重点不是给每个小函数都加锁，而是要保证：

```
node_map 查询/修改
+
链表 remove
+
链表 put_front
+
淘汰尾节点
```

这一整组操作对其他线程看起来是“原子的”。

```python
import threading

class Node:
    def __init__(self, key=0, value=0):
        self.prev = None
        self.next = None
        self.key = key
        self.value = value


class LRUCache:

    def __init__(self, capacity: int):
        self.dummy = Node(value=-1)
        self.dummy.prev = self.dummy
        self.dummy.next = self.dummy
        self.capacity = capacity
        self.node_map = {}
        # 上锁
        self.lock = threading.Lock()

    def get_node(self, key):
        if key not in self.node_map:
            return
        node = self.node_map[key]
        self.remove(node)
        self.put_front(node)
        return node

    def put_front(self, node):
        self.dummy.next.prev = node
        node.next = self.dummy.next
        self.dummy.next = node
        node.prev = self.dummy

    def remove(self, node):
        node.prev.next = node.next
        node.next.prev = node.prev

    def get(self, key: int) -> int:
        # 上锁
        with self.lock:
            node = self.get_node(key)
            return node.value if node else -1

    def put(self, key: int, value: int) -> None:
        # 上锁
        with self.lock:
            node = self.get_node(key)
            if node:
                node.value = value
                return
            node = Node(key, value)
            self.node_map[key] = node
            self.put_front(node)
            if len(self.node_map) > self.capacity:
                last_node = self.dummy.prev
                self.remove(last_node)
                del self.node_map[last_node.key]


# Your LRUCache object will be instantiated and called as such:
# obj = LRUCache(capacity)
# param_1 = obj.get(key)
# obj.put(key,value)

```

**含TTL的**

设置全局ttl

```python
import threading
import time


class Node:
    def __init__(self, key=0, value=0, expire_at=None):
        self.prev = None
        self.next = None
        self.key = key
        self.value = value
        self.expire_at = expire_at


class LRUCache:
    def __init__(self, capacity: int, ttl: float):
        self.capacity = capacity
        self.ttl = ttl

        self.node_map = {}

        self.dummy = Node()
        self.dummy.prev = self.dummy
        self.dummy.next = self.dummy

        self.lock = threading.Lock()

    def _remove(self, node):
        node.prev.next = node.next
        node.next.prev = node.prev

    def _put_front(self, node):
        first = self.dummy.next

        node.prev = self.dummy
        node.next = first

        self.dummy.next = node
        first.prev = node

    def _delete_node(self, node):
        self._remove(node)
        del self.node_map[node.key]

    def _is_expired(self, node):
        return time.monotonic() >= node.expire_at

    def _get_node(self, key):
        node = self.node_map.get(key)

        if node is None:
            return None

        # TTL 判断
        if self._is_expired(node):
            self._delete_node(node)
            return None

        # 没过期，更新 LRU 顺序
        self._remove(node)
        self._put_front(node)

        return node

    def get(self, key: int) -> int:
        with self.lock:
            node = self._get_node(key)

            if node is None:
                return -1
            # get 成功，重置 TTL
        		node.expire_at = time.monotonic() + self.ttl

            return node.value

    def put(self, key: int, value: int) -> None:
        with self.lock:
            node = self._get_node(key)

            expire_at = time.monotonic() + self.ttl

            # key 已存在
            if node is not None:
                node.value = value

                # put 时刷新 TTL
                node.expire_at = expire_at
                return

            # key 不存在 / 已过期
            node = Node(
                key=key,
                value=value,
                expire_at=expire_at
            )

            self.node_map[key] = node
            self._put_front(node)

            # 超容量
            if len(self.node_map) > self.capacity:
                last_node = self.dummy.prev
                self._delete_node(last_node)
```



如果希望：

```
LRU 操作 O(1)
+
TTL 可以主动清理
```

往往需要第二套数据结构。例如：

```
HashMap
    key -> Node

LRU Doubly Linked List
    维护访问顺序

Min Heap
    维护过期时间
```

结构变成：

```
                   ┌──────────────┐
                   │   HashMap    │
                   │ key -> Node  │
                   └──────┬───────┘
                          │
             ┌────────────┴────────────┐
             │                         │
             ↓                         ↓
      Doubly Linked List           Min Heap
       LRU访问顺序                 TTL过期顺序

     newest -> oldest          earliest -> latest
```

这时候：

```
链表：
负责 LRU

最小堆：
负责 TTL
```

最小堆堆顶永远是：

```
最早过期节点
```

所以：

```python
while heap[0].expire_at <= now:
    pop
```

就可以高效清理过期项。

### Q3：评论系统设计

> 参考短视频平台设计评论系统，需要考虑哪些内容？包括表结构设计、主键设计以及分库分表方案

短视频评论系统首先要根据主要访问模式设计数据模型。核心查询通常是按照videoId查询某个视频的一级评论和楼中楼回复，因此评论主表会包含commentId、videoId、userId、rootCommentId、parentCommentId、replyToUserId、内容、点赞数、回复数、状态和创建时间等字段。数据库不会真正构建无限层级树，而是使用一级评论加楼中楼的两级模型，通过rootCommentId快速查询某条一级评论下面的回复。

主键不能简单使用数据库自增，因为后续需要分库分表，我会使用Snowflake或者Leaf生成全局唯一、趋势递增的BIGINT评论ID，既能够保证跨库唯一，又有利于InnoDB聚簇索引写入。

分库分表方面，由于最核心查询是按videoId查评论，因此主评论表优先使用videoId作为分片键，比如8库32表，通过hash(videoId)进行路由。这样查询单视频评论可以直接定位到一个分片，不需要广播查询。

但videoId分片存在超级热门视频的数据倾斜问题，对于超热点视频可以进一步按照videoId加bucket做二级分片，将一个视频评论拆成多个桶，查询时做多路归并。普通视频不做这种处理，避免增加系统复杂度。

对于“查询用户历史评论”这种非videoId维度的访问，不会扫描所有评论分片，而是额外维护按userId分片的user_comment_index索引表。

查询评论列表使用基于createTime和commentId的游标分页，避免深分页。点赞数、回复数和评论总数不会全部实时更新MySQL热点行，而是Redis实时计数，通过Kafka异步聚合落库；热评则通过Redis ZSet维护热度排序。

> 整体原则就是：**主表围绕核心查询做分片，其他查询维度通过索引表解决；Redis承担热点和排序，Kafka处理异步事件，MySQL保存最终事实数据。**

最后可以记住这几个面试关键词：

> **两级评论模型、Snowflake主键、videoId分片、热点二级分桶、反向索引表、游标分页、Redis热数据、MQ最终一致。**
