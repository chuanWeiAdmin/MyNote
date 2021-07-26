### 一.redis的安装及其基本操作

#### 1.安装Redis

#### 2.开启Redis

- redis-service  :前台启动
- redis-service &  :后台启动

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

