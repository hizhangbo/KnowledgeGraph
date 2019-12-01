- 数据结构
  - 字符串 Strings/Blobs/位图Bitmaps
  ```
  # 值的容量最大限制512MB
  # 值可以是字符串 string，整型 int，位 bits
  # 使用场景：缓存/计数器/分布式锁
  
  # 一般场景指令
  
  set key value
  get key
  del key
  
  # 网站访问计数统计场景
  # 分布式id生成器场景
  
  # key自增1，如果不存在key，则获取值1
  incr key
  # key自减1，如果不存在key，则获取值-1
  decr key
  # key自增k，如果不存在key，则获取值k
  incrby key k
  # key自减k，如果不存在key，则获取值-k
  decrby key k
  
  # 分布式锁场景
  
  # 不管key是否存在，都设置
  set key value
  # key不存在，才设置
  setnx key value
  # key存在，才设置，更新操作
  set key value xx
  
  # 节省网络传输成本场景
  
  # 批量获取key，原子操作
  mget key1 key2
  # 批量存储
  mset key1 value1 key2 value2
  
  # 设置新值并返回旧值，原子操作
  getset key newvalue
  # 追加操作
  append key value
  # 返回字符串长度
  strlen key
  
  # 浮点类型值增减
  incrbyfloat key 3.5
  # 获取字符串指定下标范围的值
  getrange key start end
  # 设置指定下标对应的值
  setrange key index value
  ```
  - 哈希表 Hash Tables(objects)
  ```
  # key -> table 的形式存储
  
  # 一般场景指令
  # 获取hash key对应的field的value
  hget key field
  # 设置hash key对应的field的value
  hset key field
  # 删除hash key对应的field的value
  hdel key field
  
  # 获取hash key对应的所有field名称和值
  hgetall key
  # 获取hash key对应的所有field值
  hvals key
  # 获取hash key对应的所有field名称
  hkeys key
  # 判断hash key是否有field，1存在
  hexists key field
  # 获取hash key field的数量
  hlen key
  
  # 批量获取某个hash key对应的一批field值
  hmget key field1 field2 ... fieldN
  # 批量设置多个field值
  hmset key field1 value1 field2 value2 ... fieldN valueN
  
  # 用户访问记录应用场景，相较于string单个key记录的方式，这种方式使信息更加集中
  hincrby user:1:info pageview count
  
  # 缓存场景
  hmset key hashmapObject
  
  # 新建，如果已存在，则失败
  hsetnx key field value
  # 对某个field进行自增整数intCounter
  hincrby key field intCounter
  # 对某个field进行自增浮点数floatCounter
  hincrbyfloat key field floatCounter
  
  ```
  - 列表 Linked Lists
  ```
  # 有序，可重复，左右均可进行插入弹出
  # 适用时间轴功能场景
  
  # 右新增
  rpush key value1 value2 value3 ... valueN
  # 左新增（逆序）
  lpush key value1 value2 value3 ... valueN
  # 在list指定的值前|后插入newValue
  linsert key before|after value newValue
  # 左删除
  lpop key
  # 右删除
  rpop key
  # 删除指定个数元素，count=0删除所有，count>0从左到右，count<0从右到左
  lrem key count value
  # 截取范围
  ltrim key start end
  # 获取范围内的值(包含end)
  lrange key start end
  # 获取指定索引的item
  lindex key index
  # 获取列表长度
  llen key
  # 更新指定索引的值
  lset key index newValue
  
  # 消息队列场景
  
  # lpop阻塞版本，timeout=0永不阻塞
  blpop key timeout
  # rpop阻塞版本，timeout=0永不阻塞
  brpop key timeout
  
  # Stack 实现
  LPUSH + LPOP
  # Queue 实现
  LPUSH + RPOP
  # Capped Collection(固定大小列表)
  LPUSH + LTRIM
  # Message Queue
  LPUSH + BRPOP
  
  ```
  - 集合 Sets
  ```
  # 无序，不可重复，集合间操作
  
  # 添加，如果存在，则添加失败
  sadd key element
  # 删除
  srem key element
  # 随机弹出一个元素（删除）
  spop key
  # 集合元素数量
  scard key
  # 元素是否存在集合中
  sismember key element
  # 随机获取count个元素
  srandmember key count
  # 取出所有元素
  smembers key
  
  # 抽奖系统应用场景
  # 标签应用场景
  
  # 集合间操作
  # 差集
  sdiff key1 key2
  # 交集
  sinter key1 key2
  # 并集
  sunion key1 key2
  # 结果另存为resultKey
  sdiff|sinter|sunion + store resultKey
  
  ```
  - 有序集合 Sorted Sets
  ```
  # 结构 key -> score,value
  
  # 添加
  zadd key score1 element1 score2 element2
  # 删除
  zrem key element1 element2
  # 获取分数
  zscore key element
  # 分数自增
  zincrby key increScore element
  # 获取元素总数
  zcard key
  # 获取排名（score从小到大）
  zrank key element
  # 获取排名范围内的元素，并带分数
  zrange key start end [withscores]
  # 获取分数范围内的元素，并带分数
  zrangebyscore key minScore maxScore [withscores]
  # 获取分数范围内的元素个数
  zcount key minScore maxScore
  # 删除排名范围内的元素
  zremrangebyrank key start end
  # 删除分数范围内的元素
  zremrangebyscore key minScore maxScore
  
  # 排名从高到低
  zrevrank
  # 排名从高到低的范围
  zrevrange
  # 分数从高到低的排名
  zrevrangebyscore
  # 交集
  zinterstore
  # 并集
  zunionstore
  
  # 排行榜场景
  # score:timeStamp saleCount followCount
  
  
  ```
  - 超小内存唯一值计数 HyperLogLog
  ```
  
  ```
  - 地理信息位置 GEO
  ```
  
  ```
- 特性
 - 速度快：10w OPS;内存存储;C语言;单线程
 - 持久化：异步存储到硬盘
 - 多种数据结构
 - 支持多种编辑语言
 - 功能丰富：发布订阅/Lua脚本/事务/pipeline
 - 简单：不依赖外部库，单线程
 - 主从复制
 - 高可用/分布式：Redis-Sentinel/Redis-Cluster
- 通用命令
  - keys 
    ```
    # 打印所有键 O(n)
    keys *
    # 使用通配符
    keys [pattern]
    ```
  - dbsize
    ```
    # 计算key的总数 O(1)
    dbsize
    ```
  - exists key
    ```
    # 检查key是否存在 O(1)
    exists key
    ```
  - del key
    ```
    # 删除指定key-value O(1)
    ```
  - expire key seconds
    ```
    # key在seconds秒后过期 O(1)
    expire key seconds
    # 查看key剩余的过期时间 -2不存在 -1无过期时间 O(1)
    ttl key
    # 去掉key的过期时间 O(1)
    persist key
    ```
  - type key
    ```
    # 返回key的类型 string/hash/list/set/zset/none O(1)
    
    ```
  - 数据结构的内部编码
    - redisObject
      - type encoding 数据类型及其编码方式
        - string
          - raw
          - int
          - embstr
        - hash
          - hashtable
          - ziplist （节省内存）
        - list
          - linkedlist
          - ziplist
        - set
          - hashtable
          - intset
        - zset
          - skiplist
          - ziplist
      - ptr 数据指针
      - vm 虚拟内存
      - 其他信息
- 单线程
  - 一次只运行一条命令，避免执行长命令
  - 内存
  - 非阻塞IO
  - 避免线程切换和竞态消耗
- 应用场景
  - 缓存
  - 计数器（评论转发数，评论回复数）
  - 消息队列（发布订阅/阻塞订阅）
  - 排行榜（有序集合）
  - 社交网络（共同关注的人，时间轴列表）
  - 实时系统（垃圾邮件过滤）
- 特征对比
  - hash vs string （用于缓存）
    - | 命令 | 优点 | 缺点 |
      | :-----| :---- | :---- |
      | string 单key存储对象 | 编程简单，相较多key方式节约内存 | 序列化开销，不可更新部分 |
      | string 多key存储对象 | 直观，部分更新 | 内存占用大，key较为分散 |
      | hash 存储对象 | 直观，节省空间，部分更新 | 编程较string复杂，单属性ttl（过期时间）不好控制 |
  - set vs sort set vs list
    - | 类型 | 重复性 | 有序性 | 元素结构 |
      | :-----| :---- | :---- | :---- |
      | set | 不可重复 | 无序 | element |
      | sort set | 不可重复 | 有序 | element + score |
      | list | 可重复 | 有序 | element |
- 特性
  - 慢查询
    - 重要参数
    ```
    # 慢查询执行时间定义（微秒）10毫秒，不宜过大，建议1ms（执行时间）
    slowlog-log-slower-than=10000
    # 慢查询记录队列大小定义，不宜过小，建议1000
    slowlog-max-len=128
    ```
    - 动态配置命令
    ```
    config set slowlog-max-len 1000
    config set slowlog-log-slower-than 1000
    ```
    - 慢查询命令查询
    ```
    # 显示慢查询
    slowlog list
    # 获取慢查询队列
    slowlog get [n]
    # 获取慢查询队列长度
    slowlog len
    # 清空慢查询队列
    slowlog reset
    ```
    - 持久化到Mysql
  - pipeline
    - 减少网络通信次数
    - 批处理，非原子操作
    - 只能作用在一个redis节点上
  - 发布订阅
    - 发布者 publisher
    - 订阅者 subscriber
    - 频道 channel
    ```
    # 无法订阅历史消息
    # 发布，返回订阅者数
    publish channel message
    # 订阅，一个或多个
    subscribe [channel]
    # 取消订阅
    unsubscribe [channel]
    # 订阅模式
    psubscribe [pattern...]
    punsubscribe [pattern...]
    # 列出至少一个订阅者的频道
    pubsub channels
    # 获取频道订阅者数
    pubsub numsub [channel]
    ```
  - Bitmap(位图)
    - 节省内存，但需要计算位移
    - type=string 最大512MB
    - 命令
    ```
    # 设置位图指定索引的值
    setbit key offset value
    # 获取位图指定索引的值
    getbit key offset
    # 获取位值为1的个数
    bitcount key [start end]
    
    bitop op destkey key [key...]
    bitpos key targetBit [start end]
    ```
  - hyperloglog
    - type=string
    - 错误率 0.81%
    - 命令
    ```
    # 新增
    pfadd key element [element...]
    # 统计个数
    pfcount key [key...]
    # 合并
    pfmerge destkey sourcekey [sourcekey]
    ```
  - GEO（3.2版本以后）
    - type geoKey=zset
    - 命令
    ```
    # 新增
    geoadd key longitude latitude member
    # 获取
    geopos key member [member...]
    # 计算距离 unit:m(米) km(千米) mi(英里) ft(尺)
    geodist key member1 member2 [unit]
    # 范围计算
    georadius 
    ```
- 持久化
  - |命令|RDB|AOF|
    |----|:---:|:---:|
    |优先级|低|高|
    |体积|小|大|
    |恢复速度|快|慢|
    |数据安全性|丢|根据策略决定|
    |轻重|重|轻|
  - RDB DUMP 主从同步 二进制文件
    - save 同步 阻塞客户端
    - bgsave -> fork() 部分异步 消耗内存
    - 修改配置 save seconds changes (不好控制)
  - AOP LOG 日志追加命令
    - 追加策略
      - always
      - everysec(丢失一秒数据)
      - no
    - 重写策略rewrite
      - bgrewriteaof
        - fork() 回溯
      - config
        - auto-aof-rewrite-min-size # aof 文件大小触发重写
        - auto-aof-rewrite-percentage # aof 文件增长率
    - AOF阻塞
      - info Persistence # aof_delayed_fsync 阻塞次数
  - fork子进程
    - 同步操作
    - 内存越大，耗时越长
    - info Persistence # last_fork_usec 最后一次执行用时
- master-slaver
  - slaveof no one # 脱离集群模式
  - slaveof host port # 作为host:port的从节点，会清除该从节点所有数据
  - offset 对比主从同步数据一致性
- redis sentinel
  - sentinel monitor 
  - sentinel down-after-milliseconds
  - sentinel parallel-syncs
  - sentinel failover-timeout





















