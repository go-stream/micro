```
连接命令
查看服务器是否运行
ping message
echo message

验证数据库密码，连接时使用redis-cli，但数据库设置了密码，执行一般命令提示
(error) NOAUTH Authentication required.
auth password  相当于连接后再验证密码，验证后权限恢复
可以使用 ./redis-cli -a password  连接时完成验证

config set requirepass 123456  设置密码
config get requirepass               获取密码

切换数据库
select index

关闭连接
quit
exit



---------------------------------------------------------------------------------


键key-->键值存取
    set key value
    get key
键key-->键值删除
    del key
键key-->判断有无
    exists key
键key-->键值列表
    keys pattern
键key-->键值重命名
    RENAME key newkey
    RENAMENX key newkey




键key-->随机取键
    randomkey
键key-->键值序列化
    dump key
键key-->查看类型
    type key
键key-->键值移库
    move key db
键key-->键值超时
    expire key seconds
    expireat key timestamp    时间戳10位数字
    pexpire key milliseconds
    pexpireat key milliseconds-timestamp   时间戳13位数字


键key-->查询过期
    TTL  key
    PTTL key

键key-->取消过期
    persist key

--------------------------------------------------------

字符串string-->字符串值设置
    set key value
 getset key value     与set key value几乎相同，只是返回值为key的旧值，如果是新键则返回nil
    setnx key value
    mset key value [key value...]
    msetnx key value [key value...]

字符串string-->字符串获取
get key
mget key1 [key2....]

字符串string-->获取子串
getrange key start end
注：索引从0开始计算，取值包含start和end索引的字母

字符串string-->设置子串
setrange key offset value
从指定位置开始设定子串，覆盖方式，索引从0开始
例如：key yuanchunxu
setrange key 4 zhang
get key     ==>    yuanzhangu

字符串string-->获取二进制某一位
getbit key offset
索引位置从0开始，从左到右顺序，单字符共8位
例1：字符0，编码48，二进制00110000
getbit key 0   ==> 0
getbit key 1   ==> 0
getbit key 2   ==> 1
getbit key 3   ==> 1
getbit key 4   ==> 0
getbit key 5   ==> 0
getbit key 6   ==> 0
getbit key 7   ==> 0
例2：字符串10，编码49，48，二进制00110001  00110000
getbit key 0   ==> 0
getbit key 1   ==> 0
getbit key 2   ==> 1
getbit key 3   ==> 1
getbit key 4   ==> 0
getbit key 5   ==> 0
getbit key 6   ==> 0
getbit key 7   ==> 1
getbit key 8   ==> 0
getbit key 9   ==> 0
getbit key 10 ==> 1
getbit key 11 ==> 1
getbit key 12 ==> 0
getbit key 13 ==> 0
getbit key 14 ==> 0
getbit key 15 ==> 0

字符串string-->设置二进制某一位
setbit key offset value
对照获取二进制某一位进行操作

字符串string-->获取字符串长度
    strlen key

字符串string-->设置字符串及过期时间
    setex key seconds value
    psetex key milliseconds value

字符串string-->字符串运算
    incr key
    incrby key increment
    incrbyfloat key increment
    decr key
    decrby key decrement
    append key value


-------------------------------------------------------
哈希hash
存储上限：2的32次方减1    40多亿
                        |- field1 ----- val1
key   ====>>>  |- field2 ----  val2
                        |- field3 ----  val3

哈希hash-->设置字段和值
hset key field value
hsetnx key field value
hmset key field1 value1 [field2 value2...]
没有hmsetnx命令

哈希hash-->获取字段个数
hlen key

哈希hash-->获取字段和值
hget key field
hmget key field1 [field2...]

哈希hash-->获取所有字段和值
hgetall key
hkeys key    ==>field列表
hvals key     ==>每个field对应的内容列表

哈希hash-->删除字段和值
hdel key field1 [field2...]

哈希hash-->判断字段有无
hexists key field

哈希hash-->字段运算
hincrby key field increment
hincrbyfloat key field increment

哈希hash-->迭代键值对
hscan key cursor [match pattern] [count count]
cursor：整数即可，可以是负数，不影响结果
注意：count无效
示例：sscan myset1 0 match h*

--------------------------------------------------------------------
列表list
           key
           |
-----> list head <------
可以从头部（左侧）和尾部（右侧）插入数据
存储上限：2的32次方减1    40多亿

列表list-->插入数据
lpush key value1 [value2...]
lpushx key value   ==> key必须先创建
rpush key value1 [value2...]
rpushx key value  ==> key必须先创建
linsert key before | after pivot value
例：LINSERT list1 BEFORE "bar" "yes" 在bar的前面插入yes

列表list-->列表长度
llen key

列表list-->获取数据
lindex key index    ==> 正整数 从头获取；负整数 从尾获取
lrange key start stop   ==> 包括start和stop本身，正数代表从头部往尾部数，负数代表从尾部往头部数

列表list-->设置数据
lset key index value

列表list-->移除数据
lrem key count value   ---> 必须指定值，指定错误，移除失败
ltrim key start stop      ---> 删除除了指定的元素，从头（左）指定

列表list-->移除并获取数据
lpop key   -->移除并获取列表第一个元素
rpop key   -->移除并获取列表最后一个元素
blpop key1 [key2] timeout
brpop key1 [key2] timeout

列表list-->换队操作
rpoplpush source destination
brpoplpush source destination timeout

--------------------------------------------------------------------
集合set
Redis的Set是string类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据。
Redis 中 集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。
存储上限 2的32次方减1，40亿+
key <----------->{val1, val2, val3, val4,...valn}

集合set-->添加集合成员
sadd key member1 [member2, member3...]

集合set-->获取成员数量
scard key

集合set-->成员有无
sismember key member

集合set-->获取成员
smembers key
srandmember key [count]  随机返回一个或多个成员

集合set-->移除集合元素
srem key member1 [member2]

集合set-->移除并获取成员
spop key

集合set-->迭代集合元素
sscan key cursor [match pattern] [count count] 
注意：count目前无效

集合set-->成员转移
smove source destination member

{1,3,5,6}    S1
{1,3,7,8}    S2

S1 - S2  = {5, 6}
S2 - S1 = {7, 8}
集合set-->集合运算（差，并，交）
sdiff key1 [key2]        求差集
sdiffstore destination key1 [key2] 差集存储
sinter key1 [key2]      求交集
sinterstore destination key1 [key2] 交集存储
sunion key1 [key2]    求并集
sunionstore destination key1 [key2] 并集存储


--------------------------------------------------------------
有序集合
有序集合和集合一样也是string类型元素的集合,且不允许重复的成员。
每个元素都会关联一个double类型的分数
通过分数来为集合中的成员进行从小到大的排序。
有序集合的成员是唯一的,分数(score)可以重复。
集合是通过哈希表实现的，每个集合的存储上限：2的32次方

                           key
                            |
|---------------------------------------------|
score1  score2 score3 score4 score5
value1  value2  value3 value4 value5

有序集合set-->添加成员
zadd key score1 member1 [score2 member2]

有序集合set-->获取成员总数
zcard key

有序集合set-->限制条件获取成员数量
zcount key min max  获取在指定区间分数的成员数
zlexcount key min max 在有序集合中计算指定字典区间内成员数量
注意：此命令加入的值本身必须拥有明确的字典顺序
如：zadd set1 1 a 2 b 3 c 4 d 5 e 6 f 7 g
zadd set2 1 aa 2 bb 3 cc 4 ee 5 hh 6 zz
zadd set3 1 aa 3 ab 4 ac 5 ad 6 az
zadd set4 1 za 2 zb 3 zc 4 zd 5 zf



有序集合set-->获取成员分数
zscore key member

有序集合set-->获取成员索引
zrank key member   获取顺序索引
zrevrank key member 获取倒数的索引
注意：不是分数

有序集合set-->顺序获取成员数据
zrange key start stop [withscores]
例1：zrange set1 0 2   仅显示字符串
例2：zrange set1 0 2 withscores  显示分数和字符串

zrangebylex key min max [limit offset count]
例：zrangebylex set7 [a [z limit 1 3  从符合条件的索引1开始取出3个

zrangebyscore key min max [withscores][limit]
例：zrangebyscore set7 0 3 withscores limit 0 2

有序集合set-->逆序获取成员数据
zrevrange key start stop [withscores]  按分数从高到低输出
start 和 stop 指的是索引
zrevrangebyscore key max min [withscores]  按分数从高到低输出
max 和 min 指的是分数


有序集合set-->迭代获取成员数据
zscan key cursor [match pattern]


有序集合set-->删除成员
zrem key member [member ...]
zremrangebylex key min max
zremrangebyrank key start stop 
zremrangebyscore key min max

有序集合set-->分数运算
zincrby key increment member  增加指定的分数，支持负数和小数，负数即减法

有序集合set-->集合运算（并，交）
zinterstore destination numkeys key1 [key2...] 计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合
zunionstore destination numkeys key [key...] 计算给定的一个或多个有序集合的并集，并存储在新的有序集合中。


------------------------------------------------------------------------------
HyperLogLog
是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。
在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基 数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。
但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。

什么是基数?
比如数据集 {1, 3, 5, 7, 5, 7, 8}， 那么这个数据集的基数集为 {1, 3, 5 ,7, 8}, 基数(不重复元素)为5。 基数估计就是在误差可接受的范围内，快速计算基数。

pfadd key element [element ...] 添加指定元素到HyperLogLog中
pfcount key [key ...] 返回给定HyperLogLog的基数值
pfmerge destkey sourcekey [sourcekey ...] 将多个HyperLogLog合并为一个HyperLogLog

-------------------------------------------------------------------------------
获取所有配置信息命令：config get *
获取指定配置信息命令：config get 配置项名称
常规配置
requirepass foobared

./redis-cli 
客户端连接可以成功，任何操作均失败
(error) NOAUTH Authentication required.
应使用
./redis-cli -a foobared

redis设置-->绑定主机地址（bind）
bind 127.0.0.1   仅本机可以连接
bind绑定的是服务器自身的IP，一个服务器可能有多块网卡，也就有多个IP，不要误认为是指定可以连接过来的客户端IP。


redis设置-->设置服务端口
port 6379
6379作为默认端口，因为6379在手机按键上MERZ对应的号码，而MERZ取自意大利歌女Alessia Merz的名字，作者是她的歌迷。Alessia Merz的照片链接
http://image.baidu.com/search/index?tn=baiduimage&ps=1&ct=201326592&lm=-1&cl=2&nc=1&ie=utf-8&word=%E6%84%8F%E5%A4%A7%E5%88%A9%E6%AD%8C%E5%A5%B3Alessia%20Merz

redis设置-->设置客户端最大连接数
maxclients 0   表示不限制。

redis设置--设置空闲时间关闭连接
timeout 300       #0



redis设置--运行方式（daemonize）
daemonize no  非守护进程   yes  守护进程

redis设置--指定本地数据库存放目录
dir ./   与redis-server同目录

redis设置--指定本地数据库文件名
dbfilename dump.rdb

redis设置--日志级别设置
loglevel 级别标识，一共四个级别。
debug：用于debug，所有信息均记录。
verbose：冗长的，啰唆的，累赘的。除用于调试外的其它所有信息。
notice：仅记录值得注意的内部信息。
warning： 仅记录出现警告的信息。

redis设置--日志记录方式
logfile ""   标准输出，如果配合redis以守护进程方式运行，则日志发送到 /dev/null，即不记录日志，否则直接输出到控制台窗口。



redis设置--压缩设置
rdbcompression yes  默认为压缩，所发LZF

redis设置--设置数据库数量
databases 16  小于1时，redis无法使用，如果用一个16数据库建立的数据库文件，换用少于16的设置，那么原来的数据库文件将无法正常加载。但换用一个大于16的设置，那么可以正常加载使用。

redis设置-->指定存盘策略
save <secodes> <changes>
save “”   为不存盘
默认配置：
save 900 1       900秒更新1次
save 300 10     300秒更新10次
save 60 10000  60秒更新10000次

redis设置-->是否写操作后进行日志记录
appendonly no   不记录
Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内只存在于内存中。
aof文件和rdb文件同时生效，但aof的持久性更好，数据更全。

redis设置-->指定更新日志文件名，默认为appendonly.aof
appendfilename appendonly.aof

redis设置-->设定日志更新策略
appendfsync everysec
 no：表示等操作系统进行数据缓存同步到磁盘（快）
always：表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全）
everysec：表示每秒同步一次（折衷，默认值）

-----------------------------------------------------------------
高级配置


redis设置-->设置最大内存
maxmemory <bytes>
Redis在启动时会把数据加载到内存中，达到最大内存后，Redis会先尝试清除已到期或即将到期的Key，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis新的vm机制，会把Key存放内存，Value会存放在swap区

配置包含-->设置路径
include /path/to /local.conf

主从配置-->主服务ip端口配置
slaveof  <masterip> <masterport>
不能设置互为主从，可以是一主多从。设置为从服务的redis，将成为只读数据库，无法通过客户端操作写入。

主从配置-->设置主服务密码
masterauth <master-password>

虚拟内存-->是否启用虚拟内存机制
vm-enabled no/yes

虚拟内存-->设置虚拟内存文件路径
vm-swap-file /tmp/redis.swap

虚拟内存-->设置虚拟内存使用策略
vm-max-memory 0
将所有大于vm-max-memory的数据存入虚拟内存,无论vm-max-memory设置多小,所有索引数据都是内存存储的(Redis的索引数据 就是keys),也就是说,当vm-max-memory设置为0的时候,其实是所有value都存在于磁盘。默认值为0

虚拟内存-->设置页大小
vm-page-size 32
swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page上不能被多个对象共享，vm-page-size是要根据存储的 数据大小来设定的，如果存储很多小对象，page大小最好设置为32或者64bytes；如果存储很大大对象，则可以使用更大的page，如果不 确定，就使用默认值

虚拟内存-->设置swap文件page数
vm-pages 134217728

虚拟内存-->设置swap文件的线程数
vm-max-threads 4
不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。

-----------------------------???????????
哈希设置-->是否激活重置哈希
activerehashing yes

哈希设置-->指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法
hash-max-zipmap-entries 64
hash-max-zipmap-value 512

-----------------------------???????????

---------------------------------------------------------------------
发布-订阅
                                      |-------channel1------------ clientA
client1 ------channel1---------server |-------channel1------------ clientB
                                      |-------channel1------------ clientC



设计模式：观察者模式

1. 订阅
subscribe channel1 [channel2 ...]  订阅一个或多个频道
psubscribe pattern1 [pattern2 ...]  订阅一个或多个给定模式的频道

2. 发布
publish channel message

3. 退订
unsubscribe [channel 1 channel2...]
punsubscribe [pattern1 pattern2...]

4.查看订阅与发布系统状态
pubsub subcommand [argument argument...]

---------------------------------------------
 事务
Redis 事务可以一次执行多个命令， 并且带有以下两个重要的保证：
事务是一个单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。
事务是一个原子操作：事务中的命令要么全部被执行，要么全部都不执行。
一个事务从开始到执行会经历以下三个阶段：
开始事务。
命令入队。  注意：命令必须都必须正确且必须正确执行，否则事务失败。
执行事务。

事务-->开始事务
multi

事务-->执行事务
exec

事务-->取消事务
discard

事务-->监视一个或多个key
watch key1 [key1 ...] 
如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。
在非事务状态下，不会被打断
使用场合，防止某些敏感值被事务误操作

事务-->取消watch命令对所有key的监视
unwatch


----------------------------------------------------------------------
脚本
支持lua脚本
Redis 脚本使用 Lua 解释器来执行脚本。 Reids 2.6 版本通过内嵌支持 Lua 环境。执行脚本的常用命令为 EVAL。

---------------------------------------------------------------------
备份与恢复
备份命令-->save   bgsave (后台执行备份)
该命令将在 redis 安装目录中创建dump.rdb文件。

恢复数据库
需将备份文件 (dump.rdb) 移动到 redis 安装目录并启动服务即可。
注意：可以通过 config get dir  获取数据库文件应该存放的位置。
           可以通过 config get dbfilename 获取数据库文件的具体名称。

--------------------------------------------------------------------
客户端命令

客户端-->设置客户端名称
client setname yuan

客户端-->获取客户端名称
client getname 

客户端-->客户端列表
client list
id=2 addr=127.0.0.1:36574 fd=8 name=yuan age=778 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=client

客户端-->挂起客户端
client pause 毫秒数
client pause 10000 挂起10秒，10秒内客户端为阻塞态，无法执行任何命令

客户端-->杀死客户端
client kill 127.0.0.1:36574
注意：杀死与exit和quit的退出不同，还会停留在连接提示状态，只需要重新验证密码，即：auth 密码即可重新进入正常状态。

----------------------------------------------------------------------------
```



