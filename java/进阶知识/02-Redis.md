### 一.redis的安装及其基本操作

#### 1.安装Redis

#### 2.开启Redis

- redis-service  :前台启动
- redis-service &  :后台启动
- ps -ef | grep redis  查看redis是否启动

#### 3.关闭Redis

- redis-cli shutdown  Redis命令中 的关闭（推荐）
- kill -9  PID  强制关闭

#### 4.通过客户端连接Redis

- redis-cli   : 默认本机 6379端口的redis
- redis-cli  -p  6380   :  连接本地的6380端口号的redis
- redis-cli  -h 10.10.10.10  -p 6380  : 连接指定IP地址的指定端口号的redis

#### 5.一些基本操作

| 命令             | 操作                                   |
| ---------------- | -------------------------------------- |
| redis-beanchmark | 测试redis的性能（可以在任意目录使用）  |
| ping             | 测试redis是否连通  -->连接正常返回pong |
| info             | 查看redis的统计信息                    |
| select   index   | 切换数据库实例                         |
| dbsize           | 查看数据库中key 的**个数**（只看个数） |
| key  *           | 查看当前实例中所有的key信息            |
| flushdb          | 清空当前实例中所有数据                 |
| flushall         | 清空所有实例中的所有数据               |
| config get *     | 查看配置信息                           |
|                  |                                        |

### 二.Redis中 key 的命令

| 命令                  | 操作                                                   |
| --------------------- | ------------------------------------------------------ |
| key  *                | 查看数据中所有 key                                     |
|                       | \*   匹配 0 个 或多个字符  keys  k*o                   |
|                       | ?  匹配一个字符  keys  h?o                             |
|                       | [    ]  匹配其中的一个字符    keys h[abc]llo           |
| exists  key           | 判断key是否存着于当前数据库实例中                      |
| move  key  index      | 将当前实例中的key移动到另一个实例中                    |
| ttl  key              | 查看当前key的剩余生存时间                              |
|                       | （返回  -2 不存在当前key，   返回-1 没有设置生存时间） |
| expire   key   10     | 设置这个  key  的生存时间为  10 s                      |
| type key              | 查看当前  key  的数据类型                              |
| rename  key1  key2    | 将  key1  重命名为   key2                              |
| del  key1  [key2... ] | 删除指定的  key  可以是多个  返回删除的数据量          |
|                       |                                                        |

### 三.五中数据结构及操作命令

#### 1.string 类型数据的操作命令

| 命令                                 | 操作                                                    |
| ------------------------------------ | ------------------------------------------------------- |
| set  key  value                      | 将字符串设置到数据库中，如果key相同则覆盖               |
| get  key                             | 获得 key  对应的value                                   |
| append key value                     | 向指定  key  中追加value  返回追加后的长度              |
| strlen  key                          | 获得字符串的长度                                        |
| incr  key                            | 将字符串  +1  运算，不存在的话，先设置一个默认为0再  +1 |
| incrby  key  10                      | 将字符串  +10  运算                                     |
| decr  key                            | 将字符串进行  -1  运算                                  |
| decrby  key  10                      | 将字符串进行   -10  运算                                |
| getrange  key  startIndex   endindex | 截取key中start到end的字符串(闭区间，可以为负数)         |
| setrange  key  startindex  value     | 以value覆盖start开始的字符串(能覆盖几个就覆盖几个)      |
| setex  key  10  value                | 设置最大的生命周期是10s                                 |
| setnx  key  value                    | 当  key  不存在时才能设置成功                           |
| mset  k1  v1  k2 v2                  | 批量将string类型的数据设置到数据库中                    |
| mget  k1  k2                         | 批量获取                                                |
| msetnx   k1  v1  k2 v2               | 批量设置字符串，(只要有一个存在，就全部放弃)            |
|                                      |                                                         |

#### 2.List 类型数据的操作命令

| 命令                                            | 操作                                        |
| ----------------------------------------------- | ------------------------------------------- |
| lpush  key  v1  v2                              | 将多个值依次插入到**表头**（左侧）          |
| lrange  key  startIndex  endindex               | 获取指定元素区间的元素                      |
| rpush  key  v1  v2                              | 将多个值依次插入到**表尾**（右侧）          |
| lpop  key                                       | 从表头数据开始删除并返回                    |
| rpop  key                                       | 从表尾数据开始删除并返回                    |
| lindex  key  index                              | 获取指定列表的指定下标的元素                |
| llen  key                                       | 获取列表的长度                              |
| lrem  key  count  value                         | 移除列表中和  value 相等的元素              |
|                                                 | count >  0  移除从左侧开始的  count  个数据 |
|                                                 | count <  0  移除从右开始的  count 个数据    |
|                                                 | count =  0  移除所有和 value 相等的元素     |
| ltrim  key  startindex endindex                 | 在指定列表的指定下标之间组成新元素          |
| lset  key  index  value                         | 将指定列表中的指定下标设置为value           |
| linsert  key  before/after  flagvalue  newvalue | 插入指定元素，在目标value之前或者之后       |
|                                                 |                                             |

#### 3.set 类型数据的操作命令

| 命令                        | 操作                                              |
| --------------------------- | ------------------------------------------------- |
| sadd  key  v1  v2           | 将多个值加入到指定的集合中(有重复数据自动忽略)    |
| smembers  key               | 获取指定集合中全部的元素                          |
| sismember  key  value       | 判断元素是否在指定的集合中  (存在：1 ，不存在：0) |
| srem  key  value1  value2   | 移除指定的元素(不存在的会自定忽略)                |
| srandmember   key  [count ] | 从集合中随机选取一个或多个                        |
|                             | count  >  0  随机选取的多个元素不重复             |
|                             | count  <  0  随机选取的多个元素可能重复           |
| spop  key  [ count ]        | 随机移除一个或者多个                              |
| smove   k1  k2  member      | 把元素从  k1  移动到  k2                          |
| sdiff  k1  k2  k3           | 获取 k1  中有，其他集合中没有的                   |
| sinter  k1  k2  k3          | 获取所有集合中都有的元素                          |
| sunion  k1  k2  k3          | 获取所有集合，所有元素组成的集合                  |
|                             |                                                   |

#### 4.hash 类型数据的操作命令

| 命令                                         | 操作                                              |
| -------------------------------------------- | ------------------------------------------------- |
| hset  key  filde1  value   filed2  value     | 将一个或者多个filed-value设置到哈希表中           |
| hget  key  filed                             | 获取指定哈希表的制定filed                         |
| hmset  key  filed1   value1  filed2   value2 | 批量同1功能相同(用的不多)                         |
| hmget  key  field1 field2                    | 批量获取指定哈希表的field值                       |
| hgetall  key                                 | 获取指定哈希表中的全部数据                        |
| hdel  key  field1  field2                    | 从指定哈希表中删除一个或多个                      |
| hlen  key                                    | 获取哈希表中field 的个数                          |
| hexists  key  field                          | 判断哈希表中是否存着field  （存着：1，不存在：0） |
| hkeys  key                                   | 获取key中所有的field列表                          |
| hvalues  key                                 | 获取key中所有的value列表                          |
| hincrby  key  field  10                      | 对指定key下的指定field  +10(只能加整)             |
| hincrbyfloat  key  field  10.5               | 对指定key下的指定field  +10.5  (可以加小数)       |
| hsetnx  key  field  value                    | 当field-value存着时放弃设置                       |



#### 5.zset 类型数据的操作命令

| 命令                                        | 操作                                             |
| ------------------------------------------- | ------------------------------------------------ |
| zadd  key  score(分数)  member              | 将一个或多个元素及分数加入有序集合               |
| zrang  key  startindex  endindex            | 获取指定下标区间的元素（闭区间）                 |
| zrevrange  key   startindex  endindex       | 从大到小排序获取元素                             |
| zrangebyscore  key  min  max  [ withscore ] | 获取指定有序集合中指定元素区间的元素             |
| zrem  key  value1  value2                   | 删除指定有序集合中的一个或多个元素               |
| zcard  key                                  | 获取指定集合的长度                               |
| zcount  key  min  max                       | 获取指定区间内元素的个数                         |
| zrank  key  member                          | 获取*从小到大*指定有序集合指定元素的  **排位**   |
| zrevrank  key  member                       | 获取*从大到小*的指定有序集合指定元素的  **排位** |
| zscore  key  member(分数)                   | 获取指定元素的分数                               |

### 四.redis持久化策略

#### 1.RDB策略

注：一般开启RDB就可以

- 定义：在指定的时间间隔内，redis服务执行指定次数的写操作会自定触发一次持久化操作
- 配置文件中的内容：
  - save  <seconds>  <changes>
  - dbfilename      RDB持久数据存储文件名
  - dir    RDB持久化文件所在目录

RDB策略是默认持久化策略策略，redis服务开启的时候就已经开启了



#### 2.AOF策略

注：采用操作日志来记录进行一次写操作，每一次redis启动时，都会重新执行一次操作日志中的命令

配置文件中的内容

​	appendonly  ：  配置是否开启AOF策略

​	appendfilename  ：  配置操作日志文件

### 五.Redis 的事务

#### 1.基本概念

- 事务：把一组数据库命令放在一起执行，保证原子性，要么同时成功，要么同时失败
- redis的事务：允许把一组redis命令放在一起，把命令序列号，然后一起执行，保证部分原子性

#### 2.语句

- multi  ：用来标记一个事务的开始
- exec  ：用力执行事务队列中所有的命令
- dicard  ： 清除所有队列已经压入队列中的命令，并且解释整个事务
- watch  ： 监控某一个建，当事务在执行过程中，此键码值发生变化则放弃执行，否则正常执行
- unwatch  ：  放弃监控所有的建

#### 3.redis 保证原子性

注：redis ***只能保证部分原子性***

- 如果一组命令中，有在压入事务队列的过程中发生错误的命令，**则本事务中所有的命令都不执行**，能够保证事务的原子性
- 如果一组命令中，在压入事务队列的过程中正常，**但是在执行事务队列命令时发生了错误**，不全影响其他的命令执行，不能保证事务的原子性

### 六.Redis 的消息订阅与发布

#### 1.定义

- redis 客户端订阅频道，消息发布者往频道上发布消息，所有订阅此消息的客户端都能接收到消息

#### 2.命令

- subscribe  : 订阅一个或多个频道的消息

  subscribe  ch1  ch2

- publish  ： 将消息发布到指定的频道中

  publish  ： publish ch1  ch2

- psubscribe   :  订阅，支持通配符   psubscribe    new*



### 七.Redis的集群

#### (一).主从复制

1. 搭建三台redis服务

2. 通过redis客户端连接三台redis服务

3. 查看三台redis服务在集群中的状态角色

   - info  replication  查询主从信息
   - role:master  表示为主机
   - role:slave   表示从机
   - connected_slaves:0  表示从机的数量

4. 设置主从关系： slaveof  127.0.0.1  6379

5. 全量复制：一旦主从关系确认，会自动的把主机上的已有信息同步到从库中

6. 增量复制： 主库中数据增加会同步到从库中

7. 主写从读：读写分离

8. 主机宕机：从机原地待命

   master-Link_status  从  up  变成了 dowm

9. 主机宕机  从机上位

   - 从机断开原来的主从关系------>  slaveof  no  one 
   - 重新设置主从关系（将其它从机重新设置主机）

10. **注：任何redis服务中只要有从的身份就不能写数据**

