---
title: Redis数据库
date: 2019-01-09 22:24:13
tags: 
  - Redis
category: 数据库
---

redis 是一种 nosql 数据库，他的数据是保存在内存中，同时 redis 可以定时把内存数据同步到磁盘，既可以将数据持久化，并且比 memcached 支持更多的数据结构(string, list列表, set集合, sorted set有序集合，hash表)文档参考: "http://redisdoc.com/index.html"
<!-- more -->

## redis 使用场景
1、登录会话存储：存储在redis中，与memcached相比，数据不会丢失。
2、排行版/计数器：比如一些秀场类的项目，经常会有一些前多少名的主播排名。还有一些文章阅读量的技术，或者新浪微博的点赞数等。
3、作为消息队列：比如celery就是使用redis作为中间人。
4、当前在线人数：还是之前的秀场例子，会显示当前系统有多少在线人数。
5、一些常用的数据缓存：比如我们的BBS论坛，板块不会经常变化的，但是每次访问首页都要从mysql中获取，可以在redis中缓存起来，不用每次请求数据库。
6、把前200篇文章缓存或者评论缓存：一般用户浏览网站，只会浏览前面一部分文章或者评论，那么可以把前面200篇文章和对应的评论缓存起来。用户访问超过的，就访问数据库，并且以后文章超过200篇，则把之前的文章删除。
7、好友关系：微博的好友关系使用redis实现。
8、发布和订阅功能：可以用来做聊天软件。

## redis在ubuntu系统中的安装与启动
1、安装
```
sudo apt-get install redis-server
```
2、卸载
```
sudo apt-get purge --auto-remove redis-server
```
3、启动：redis 安装后默认自动启动，可以通过以下命令查看
```
ps aux|grep redis
```
4、停止
```
sudo service redis-server stop
```

## 对redis的操作
对redis的操作可以用两种方式，第一种方式采用redis-cli，第二种方式采用编程语言，如pyhton和Java等。
1、使用redis-cli对redis进行字符串操作

2、启动redis：
```
sudo servic redis-server start
```

3、连接 redis-server： 
```
redis-cli -h [ip] - p [端口]
```

4、添加：
```
set key value
set username threefire
```
将字符串值 value 关联到 key。如果 key 已经持有其他值，set 命令就是覆写旧值，无视其类型。并且默认的过期时间是永久，即永不过期。

5、删除：
```
del key
del username
```

6、设置过期时间：
```
expire key timeout(单位为秒)
```
也可以在设置值的时候，一同指定过期时间。
```
set key value EX timeout
或者
setex key timeout value
```

7、查看过期时间
```
ttl key
ttl username
```

8、查看当前 redis 中的所有 key：
```
keys *
```

9、列表操作:
 + 在列表左边添加元素：
```
lpush key value
```
将值 value 插入到列表 key 的表头。如果 kye 不存在，一个空列表会被创建并执行 lpush 操作。当 key 存在但不是列表类型时，将返回一个错误。
 + 在列表右边添加元素：
```
rpush key value
```
将值 value 插入到列表 key 的表尾。如果 key 不存在，一个空列表会被创建并执行 RPUSH 操作。当 key 存在但不是列表类型时，返回一个错误。
 + 查看列表中的元素：
```
lrange key start stop 
```
返回列表 key  指定区间内的元素，区间以偏移量 start 和stop 指定，如果要左边的第一个到最后一个 lrange key 0 -1。

 + 移除列表中的元素：
  - 移除并返回列表 key 的头元素
```
lpop key
```
  - 移除并返回列表的尾元素
```
rpop key
```
  - 同时移除多个并返回列表 key 的的元素
```
# 删除key这个列表中count个value
lrem key count value
```
 + 指定返回第几个元素：
```
lindex key index
```
 + 获取列表中的元素个数
```
llen key
```
+ 删除指定元素：
```
lrem key count value
```
根据参数 count 的值，移除列表中与参数 value 相等的元素。count 的值可以是以下几种：
 + count>0: 从表头开始向表尾搜索，移除与 value 相等的元素，数量为 count。
 + count<0: 从表尾开始向表头搜索，移除与 value 相等的元素，数量为 count 的绝对值。
 + count=0：移除表中所有与 value 相等的值。

10、set 集合的操作
 + 添加元素:
```
sadd set value1 value2....
如：
sadd name liang liu
```
 + 查看元素：
```
smembers set
如：
smembers name
```
 + 移除元素：可以同时移除多个值
```
srem set value...
如 srem name liang
```
 + 查看元素个数：
```
scard set
如 scard name
```
 + 获取多个集合的交集：
```
sinter set1 set2
如
sinter name1 name2
```
 + 获取多个集合的并集：
 ```
 sunion set1 set2
 如
 sunion name1 name2
 ```
 + 获取多个集合的差集：
```
sdiff set1 set2
如：
sdiff name1 name2
```

11、hash 哈希操作：
哈希表就相当于保存的字典
 + 添加一个新值
```
hset key field value
如
hset web baidu baidu.com
```
将哈希表 key 中的域 field 的值设为 value。如果 key 不存在，一个新的哈希表被创建并进行 HSET 操作。如果域 field 已经存在于哈希表中，旧值将被覆盖。
 + 获取哈希中的 field 对应的值：
```
hget key field
如：
hget web baidu
```
 + 删除 field 中的某个 field:
```
hdel key field
如：
hdel web baidu
```
 + 获取某个哈希中所有的 field 和 value：
```
hgetall key
```
 + 获取某个哈希中所有的 field：
```
hkey key
```
 + 获取某个哈希中所有的值：
```
hvals key 
```
 + 判断哈希中是否存在某个 field：
```
hexists key field
如
hexists web baidu
```
 + 获取哈希中总共的键值对：
```
hlen field
```

12、事务操作
Redis 事务可以一次执行多个命令，事务具有以下特征
 + 隔离操作：事务中的所有命令都会序列化、按顺序地执行、不会被其他命令打扰。
 + 原子操作：事务中的命令要么全部被执行，要么全部都不执行。
 + 开启一个事务：
```
multi
```
 + 执行事务：
```
exec
```
 会将在 multi 和 exec 中的操作一并提交
 + 取消事务：
```
discard
```
 会将 multi 后的所有命令取消
 + 监视一个或者多个 key：
```
watch key...
```
监视一个(或多个)key， 如果在事务执行之前这个(或这些)key 被其他命令所改动，那么事务将被打断。
 + 取消所有 key 的监视
```
unwatch
```

13、发布/订阅操作:
 + 给某个频道发布消息:
```
publish channel message
```
 + 订阅某个频道的消息

14、持久化：redis提供了两种数据备份方式，一种是RDB，另外一种是AOF，以下将详细介绍这两种备份策略：
####RDB 
默认开启，关闭：把配置文件中所有的save都注释，就是关闭了。
 + 同步机制: 可以指定某个时间内发生多少个命令进行同步。比如1分钟内发生了2次命令，就做一次同步。
 + 存储内容：存储的是redis里面的具体的值
 + 优点：
  （1）存储数据到文件中会进行压缩，文件体积比aof小。
  （2）因为存储的是redis具体的值，并且会经过压缩，因此在恢复的时候速度比AOF快。（3）非常适用于备份。
   缺点： 
  （1）RDB在多少时间内发生了多少写操作的时候就会出发同步机制，因为采用压缩机制，RDB在同步的时候都重新保存整个Redis中的数据，因此你一般会设置在最少5分钟才保存一次数据。在这种情况下，一旦服务器故障，会造成5分钟的数据丢失。
  （2）在数据保存进RDB的时候，Redis会fork出一个子进程用来同步，在数据量比较大的时候，可能会非常耗时。

####AOF 
开启：在配置文件中appendonly yes即开启了aof，为no关闭。
 + 同步机制: 每秒同步或者每次发生命令后同步
 + 存储内容：存储的是执行的更新数据的操作命令
  优点： 
 （1）AOF的策略是每秒钟或者每次发生写操作的时候都会同步，因此即使服务器故障，最多只会丢失1秒的数据。 
 （2）AOF存储的是Redis命令，并且是直接追加到aof文件后面，因此每次备份的时候只要添加新的数据进去就可以了。
 （3）如果AOF文件比较大了，那么Redis会进行重写，只保留最小的命令集合。
  缺点：
 （1）AOF文件因为没有压缩，因此体积比RDB大。 
 （2）AOF是在每秒或者每次写操作都进行备份，因此如果并发量比较大，效率可能有点慢。
 （3）AOF文件因为存储的是命令，因此在灾难恢复的时候Redis会重新运行AOF中的命令，速度不及RDB。
 参考文档：" http://redisdoc.com/topic/persistence.html#redis "

15、安全：在配置文件中，设置 requirepass password，那么客户连接的时候就需要密码。配置文件路径"etc/redis/reid.conf"。
两种方式登录：
 + 在启动时添加 "-a" 参数，后面加密码
 + 启动之后，使用"auth" 命令授权

## Python 操作redis
1、安装 python-redis:
```
pip install redis
```

2、新建一个文件比如 redis_test.py, 然后初始化一个 redis 实例变量，并且在 ubuntu 虚拟机中开启 redis。比如虚拟机的ip地址为 192.168.0.1
```
from redis import Redis

redis_test = Redis(host='192.168.0.1', port=6379, password=xxx)
```

3、对字符串的操作：操作 redis 的方法名称，与在 redis-cli 中一样
```
# 添加一个值，过期时间为60秒
redis_test.set('username', 'threefire', ex=60)
# 获取一个值
redis_test.get('username')
# 删除一个值
redis_test.del('username')
```

4、对列表操作：也是和 redis-cli 中操作一样
```
# 在列表左边添加一个值
redis_test.lpush('languages', 'python')
# 获取有一个值
redis_test.lrange('languages', 0, -1)
```

5、对集合操作
```
# 添加一个元素
redis_test.sadd('team', 'xiaoxue')
# 获取所有元素
redis_test.smembers('team')
```

6、对哈希的操作：
```
# 添加一个值
redis_test.hset('web', 'baidu', 'baidu.com')
# 获取哈希中的所有值
redis_test.hgetall('web')
```

7、事务操作
```
# 定义一个实例
pip = redis.pipeline()
pip.incr('Bank1')
pip.desc('Bank2')
# 执行事务
pip.execute()
```
