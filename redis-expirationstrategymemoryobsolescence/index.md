# Redis 底层知识 —— 过期删除 & 内存淘汰


> Redis重要的使用场景有两个 : 缓存和分布式锁。 </br>
> 这两个使用方式或多或少都涉及到两个底层知识 : 过期删除 & 内存淘汰 </br>

<!--more-->

# 过期删除策略
Redis 可以对 key 设置过期时间，因此需要相应的机制将已过期的键值对删除，而做这个工作的就是过期键值删除策略。
## 过期时间相关命令
```bash
# 设置过期时间
> expire <key> <n>
> pexpire <key> <n>
> expireat <key> <n>
> pexpireat <key> <n>

> set <key> <value> ex <n>
> set <key> <value> px <n>
> setex <key> <n> <valule>

# 查看 key 过期时间还剩多少
> ttl <key>

# 取消 key 的过期时间
> persist key
(integer) 1
```

## Redis 如何判定 key 已过期了？
每当我们对一个 key 设置了过期时间时，Redis 会把该 key 带上过期时间存储到一个过期字典（expires dict）中，也就是说「过期字典」保存了数据库中所有 key 的过期时间。</br>
Redis判定过期是通过将当前时间与过期时间进行比较来进行的。

## Redis 过期删除策略
Redis 选择「惰性删除+定期删除」这两种策略配和使用.

### 常见的过期删除策略
定时删除 : 在设置 key 的过期时间时，同时创建一个定时事件，当时间到达时，由事件处理器自动执行 key 的删除操作。</br>
惰性删除 : 不主动删除过期键，每次从数据库访问 key 时，都检测 key 是否过期，如果过期则删除该 key。</br>
定期删除 : 每隔一段时间「随机」从数据库中取出一定数量的 key 进行检查，并删除其中的过期key。</br>

# 内存淘汰策略
## 设置 Redis 最大运行内存
在配置文件 redis.conf 中，可以通过参数 maxmemory <bytes> 来设定最大运行内存

## Redis 内存淘汰策略（8种）
### 不进行数据淘汰的策略
noeviction（Redis3.0之后，默认的内存淘汰策略）: 它表示当运行内存超过最大设置内存时，不淘汰任何数据，这时如果有新的数据写入，会报错通知禁止写入，不淘汰任何数据，单纯的查询或者删除操作可以正常工作。
### 进行数据淘汰的策略
#### 过期时间的数据中进行淘汰
volatile-random：随机淘汰设置了过期时间的任意键值；</br>
volatile-lru（Redis3.0 之前，默认的内存淘汰策略）：淘汰所有设置了过期时间的键值中，最久未使用的键值；</br>
volatile-lfu（Redis 4.0 后新增的内存淘汰策略）：淘汰所有设置了过期时间的键值中，最少使用的键值；</br>
volatile-ttl：优先淘汰更早过期的键值。</br>
#### 所有数据中进行淘汰
allkeys-random：随机淘汰任意键值;</br>
allkeys-lru：淘汰整个键值中最久未使用的键值；</br>
allkeys-lfu（Redis 4.0 后新增的内存淘汰策略）：淘汰整个键值中最少使用的键值。</br>

### 如何查看当前 Redis 使用的内存淘汰策略
```bash
127.0.0.1:6379> config get maxmemory-policy
1) "maxmemory-policy"
2) "noeviction"
```


# 参考
https://www.xiaolincoding.com/redis/module/strategy.html#%E8%BF%87%E6%9C%9F%E5%88%A0%E9%99%A4%E7%AD%96%E7%95%A5
