# 前言
### 为什么我要尝试写作技术书籍
- 一个人年轻时经历的艰难会在未来成为他的财富

# 第一篇 基础和应用篇

## 1.1 授人以鱼不如授人以渔
Redis：Remote Dictionary Service，远程字典服务

### 1.1.1 由Redis面试想到的
架构师的技能水平很高，对提升团队研发效率很有帮助，我们非常钦佩和羡慕，但是普通开发者如果习惯于在架构师封装好的东西上，只专注于做业务开发，那么久而久之，在技术理解和成长上就会变得迟钝甚至麻木。从这个角度上看，架构师可能成为普通开发者的”敌人“，他的强大能力会让大家变成”温室的花朵“，一旦遇到环境变化就会不知所措

## 1.2 万丈高楼平地起 — Redis基础数据结构

### 1.2.2 5种基础数据结构
Redis有5种基础数据结构，分别为：string（字符串）、list（列表）、hash（字典）、set（集合）、zset（有序集合）
#### string（字符串）
- Redis所有的数据结构以唯一的key字符串作为名称，然后通过这个唯一key值来获取相应的value数据，不通类型的数据结构的差异就在于value的结构不一样
- Redis的字符串是动态字符串，是可以修改的字符串，内部结构的实现类似于Java的ArrayList，实际空间capacity一般高于实际字符串长度len，1M空间冬天扩容
- 键值对：set name codehole; get name; exits name; del name; get name;
- 批量键值对：set name1 codehole; set name2 holycoder; mget name1 name2 name3; mset name1 boy name2 girl name3 unkown; mget name1 name2 name3;
- 过期和set命令扩展：set name codehole; get name; expire name 5; get name; setex name 5 codehole; get name; setnx name codehole; get name; set nax name holycoder; get name;
- 计数：set age 30; incr age; incrby age 5; incrby age -5; set codehole 9223372036854775807; incr codehole;
#### list（列表）
- Redis的列表相当于Java语言里面的LinkedList，注意它是链表不是数组，意味着插入和删除操作非常快，但索引定位很慢
- Redis的列表结构常用来做异步队列使用，将需要延后处理的任务结构体序列化成字符串，塞进Redis的列表，另一个线程从这个列表中轮训数据进行处理
- 右边进左边出：队列 rpush books python java golang; llen books; lpop books;
- 右边进右边出：栈 rpush books python java golang; rpop books;
- ##### 慢操作
- index相当于Java链表中的get(int index)方法
- ltrim两个参数start_index和end_index定义了一个区间，其他的都删掉
- index可以为负数，-1位倒数第一个元素
- rpush books python java golang; lindex books 1; lrange books 0 -1; ltrim books 1 -1; lrange books 0 -1; ltrim books 1 0; llen books;
- ##### 快速列表
- Redis底层不是简单的linkedlist，而是quicklist的一个结构
- 元素较少的时候，使用一块连续内存存储，结构是ziplist，数据比较多的时候改为quicklist，多个ziplist使用双向指针串起来使用
#### hash（字典）
- Redis的字典相当于Java语言里面的HashMap，数组+联表二维结构，渐进式rehash
- hset books java "think in java"; hset books golang "concurrency in go"; hset books python "python cookbook"; hgetall books; hlen books; hget books java; hset books golang "learning go programming"; hget books golang; hmset books java "effective java" ptyhon "learning python";
- hash可对单个key进行计数：hset user-laoqian age 29; hincrby user-laoqian age 1
#### set（集合）
- Redis的集合相当于Java语言里面的HashSet，键值对无序的
- sadd books python; sadd books python; sadd books java golang; smembers books; sismember books java; sismember books rust; scard books; spop books;
#### zset（有序列表）
- 类似于Java的SortedSet和HashMap的结合体，跳跃列表数据结构
- zadd books 9.0 "think in java"; zadd books 8.9 "java concurreency"; zadd books 8.6 "java cookbook"; zrange books 0 -1; zrevrange books 0 -1; zcard books; zscore books "java concurrency"; zrank books "java concurrency"; zrangebyscore books 0 8.91; zrangebyscore books -inf 8.91 withscores; zrem books "java concurrency"; zrange books 0 -1;
- 跳跃列表，最下面一层所有的元素都会串起来。然后每隔几个元素挑选出一个代表，再将这几个代表使用另外一级指针串起来。然后在这些代表里面再挑出二级代表，再串起来。最终就形成了金字塔结构。跳跃列表采取一个随机策略来决定新元素可以兼职到第几层。L0层100%，L1层50%，L2层25%...

### 1.2.3 容器型数据结构的通用规则
list set hash zset都是容器型数据结构，共享两条规则
1. create if not exists，如果容器不存在，那就创建一个，再进行操作
2. drop if no elements，如果容器的元素没有了，那么立即删除容器，释放内存

### 1.2.4 过期时间
Redis所有的数据结构都可以设置过期时间，时间到了，Redis会自动删除相应对象

## 1.3 千帆竞发 — 分布式锁
原子操作是指不会被线程调度机制打断的操作。这种操作一旦开始，就会一直运行到结束，中间不会有任何线程切换

### 1.3.1 分布式锁的奥义
- 分布式锁本质上要实现的目标就是在Redis里面占一个”坑“，当别的进程也要占坑时，发现那里已经有一根”大萝卜“了，就只好放弃或者稍后再试
- 占坑一般使用setnx(set if not exists)指令，先来先占，用完了，再调用del指令释放”坑“
- setnx lock:codehole true; del lock:codehole;
- 一般拿到锁之后，再给锁加一个过期时间，避免占坑后逻辑出现异常，没有释放锁，导致死锁
- setnx lock:codehole true; expire lock:codehole 5; del lock:codehole;
- 如果setnx和expire之间服务器进程突然挂掉，还是会造成死锁。也不能加事务，事务的特点是一口气执行，要么全执行，要么一个不执行，setnx有可能没抢到锁，expire是不应该执行的。
- Redis 2.8版本，引入了setnx和expire指令可以一起执行
- set lock:codehole true ex 5 nx; del lock:codehole;

### 1.3.2 超时问题
- Redis分布式锁不要用于较长时间的任务
- 稍微安全的办法：将set的value参数设置为一个随机数，释放锁的时候先匹配随机数是否一致，然后再删除key。确保当前线程占的锁不会被其他线程释放，除非这个锁是因为过期而被服务器释放，但匹配和删除不是一个院子操作，需要使用Lua脚本处理，保证连续多个指令的原子性执行。这个方案只是相对安全一些，如果真的超时了，当前线程逻辑没有处理完，其他线程也会趁虚而入

### 1.3.3 可重入性
- 如果一个锁支持同一个线程的多次加锁，那么这个锁是可重用的。Redis如果要支持可重入，需要客户端对set封装，使用线程的Threadlocal变量存储当前持有锁的计数

## 1.4 缓兵之计 — 延时队列

### 1.4.1 异步消息队列
- Redis的list（列表）数据结构常用来作为异步消息队列使用，用rpush和lpush操作入队列，用lpop和rpop操作出队列

### 1.4.2 队列空了怎么办
- 如果队列空了，客户端就会陷入pop的死循环，通常我么使用sleep来解决这个问题

### 1.4.3 阻塞读
- 睡眠会导致延迟增大，blpop/brpop，阻塞读在队列没有数据的时候，会立即进入休眠状态，一旦数据到来，则立刻醒过来。消息的延迟几乎为0

### 1.4.4 空闲连接自动断开
- 如果线程一直阻塞在那里，Redis的客户端就成了空闲连接，闲置过久，服务器一般会主动断开连接，减少闲置资源占用。这个时候blpop/brpop会抛出异常，所以客户端需要捕获异常后重试

### 1.4.5 锁冲突处理
客户端加锁没成功处理策略
1. 直接抛出异常，通知用户稍后重试
2. sleep一会儿，然后重试
3. 将请求转移至延时队列，过一会再试

### 1.4.6 延时队列的实现
- 延时队列可以通过Redis的zset（有序列表）来实现，我们将消息序列化成一个字符串作为set的value，这个消息的到期处理时间做为score，然后多个线程轮训zset获取到期的任务进行处理。
- Redis的zerm方法可以返回多线程中是否抢到任务

### 1.4.7 进一步优化
- 同一个任务可能会被多个进程取到之后再使用zrem进行争抢，没抢到的进程白取了一次任务，可使用lua scripting来优化，将zrangebyscore和zrem一同挪到服务器端进行原子操作

## 1.5 节衣缩食 — 位图
位图不是特殊的数据结构，它的内容其实就是普通的字符串，也就是byte数组，我们可以使用普通的get/set直接获取和设置整个位图的内容，也可以使用位图操作getbit/setbit等将byte数组看成”位数组“来处理

### 1.5.1 基本用法
零存零取，整存零取

### 1.5.2 统计和查找
Redis提供了位图统计指令bitcount和位图查找指令bitpos

### 1.5.3 魔术指令bitfield
- bitfield有三个子指令，get、set、incrby
- bitfield提供了溢出策略子指令overflow，饱和截断，失败不执行

## 1.6 四两拨千斤 — HyperLogLog
用来解决非精确统计问题，UV

### 1.6.1 使用方法
pfadd和set集合的sadd的用法是一样的，来一个用户ID，就将用户ID塞进去就是。pfcount和scard的用法一样，直接获取计数值

### 1.6.2 pfadd中的pf是什么意思
HyperLogLog数据结构的发明人Philippe Flajolet，pf是缩写

### 1.6.3 pfmerge适合的场景
pfmerge，用于将多个pf计数值累加在一起形成一个新的pf值

### 1.6.4 注意事项
需要占据12KB的存储空间，数据少的时候采用稀疏矩阵存储，超过阈值后，才会一次性转变为稠密矩阵

### 1.6.5 HyperLogLog实现原理
给定一系列随机数，记录下低位连续零位的最大长度K，可通过K估算出随机数的数量N。K和N的对数之间存在显著的线性相关性

### 1.6.6 pf的内存占用为什么是12KB
实现的时候使用的是2的14次方桶，每个桶的maxbits需要6个bit存储

## 1.7 层峦叠嶂 — 布隆过滤器
布隆过滤器，专门解决去重问题

### 1.7.1 布隆过滤器是什么
是一个不怎么精确的set结构，当使用contains方法判断对象是否存在，可能产生误判。它说某个值存在时，可能不存在。它说某个值不存在时，那他肯定不存在

### 1.7.2 Redis中的布隆过滤器
docker run -p6379:6379 redislabs/rebloom

### 1.7.3 布隆过滤器的基本用法
- bf.add添加元素，bf.exists查询元素是否存在，添加多个元素要用bf.madd，查询多个元素是否存在bf.mexists
- bf.reserve指令显示创建，key，error_rate，initial_size

### 1.7.4 注意事项
initial_size设置过大，会浪费存储空间。error_rate越小，需要存储空间越大

### 1.7.5 布隆过滤器的原理
- 数据结构里面就是一个大型的位数组和几个不一样的无偏hash函数，无偏hash就是hash值比较平均
- add的时候进行hash，然后对长度取模，相应位置1，查询的时候，全是1代表极有可能存在，只要有一位是0，那么肯定不存在
- 实际元素大于初始化数量，应该对布隆过滤器进行重建，重新分配size更大的过滤器，并且把历史元素add进去

### 1.7.6 空间占用估计
- 预计元素数量n，错误率f=>数组的长度l，hash的最佳数量k
- set中存储的是每隔元素的内容，而布隆过滤器仅仅存储元素的指纹

### 1.7.7 实际元素超出时，误判率会怎样变化
- 错误率10%，倍数比为2，错误率会到40%
- 错误率1%，倍数为2，错误率会升到15%
- 错误率为0.1%，倍数为2，错误率会升到5%

### 1.7.8 用不上Redis 4.0怎么办
- Redis布隆过滤器Python库，pyreBloom
- Redis布隆过滤器Java库，orestes-bloomfilter

### 1.7.9 布隆过滤器的其他应用
- 爬虫过滤爬过的网站
- NoSQL，查询某个row，先通过内存中过滤器过滤大量不存在的row
- 垃圾邮件过滤

## 1.8 短尾求生 — 简单限流
除了控制流量，限流还有一个应用目的是控制用户行为，避免垃圾请求

### 1.8.1 如何使用Redis来实现简单限流策略
系统要限制用户的某个行为在指定的时间里智能发生N次

### 1.8.2 解决方案
zset数据结构的score值，通过score来保留时间窗口。每一个行为都会作为zset中的一个key保存下来，同一个用户的同一种行为用一个zset记录

## 1.9 一毛不拔 — 漏斗限流
- 漏斗的剩余空间代表着当前行为可以持续进行的数量，漏嘴的流水率代表着系统允许该行为的最大频率
- Funnel使用hash，无法保证原子性，从hash结构中取值，然后内存运算，再回填到hash。而一旦加锁，就意味着加锁失败可能，选择重试会导致性能下降，选择放弃，影响用户体验，需要Redis-Cell救星

### 1.9.1 Redis-Cell
- Redis 4.0模块提供Redis-Cell模块，命令为cl.throttle
- cl.throtttle laoqian:reply 15 30 60 1，laoqian:reply：key laoqian，15：capacity是漏斗容量，30：operations，60 seconds，30/60为漏斗速率，1：need 1 quota，可选，默认是1
- 返回值0，15，14，-1，2；0代表允许，1表示拒绝；15：漏斗容量capacity，14：漏斗剩余空间left_quota；-1：如果被拒绝了，需要多长时间后再试，单位s；2：多长时间后，漏斗完全空出来

## 1.10 近水楼台 — GeoHash

### 1.10.1 用数据库来算附近的人
一般方法都是通过指定举行区域来限定元素的数量，然后对区域内的元素进行全量距离计算再排序。数据库表需要把经纬度坐标加上双向复合索引（x, y）

### 1.10.2 GeoHash算法
- GeoHash可以将二维的经纬度坐标映射到一维的整数
- 地球看为平面，二分法划分方格，00，01，10，11，Redis里面经纬度用52位整数进行编码，放进zset中，zset的value元素的key，score是GeoHash的52位整数

### 1.10.3 Geo指令的基本用法
- 增加，geoadd，集合名称，多个经纬度名称三元组
- 距离，geodist，集合名称、两个名称和距离单位
- 获取元素位置，geopos，集合，元素名称，获取的坐标是有损的
- 获取元素的 hash值，base32编码
- 附近的公司，georadiusbymember，查询指定元素附近的其他元素；georadius，查询附近的的元素指令
- 数据量过大，需要对Geo数据进行拆分，按照国家拆分、省、市、区

## 1.11 大海捞针 — scan
- 如何从海量的key中找出满足特定前缀的key列表
- keys命令用来列出所有满足特定正则字符串规则的key
- 指令缺点：1. 没有offset、limit 参数 2. keys算法是遍历，复杂度O(n)，这个指令卡顿，所有读写Redis其他指令都会延后甚至报错，因为Redis单线程，引入了scan命令解决
- scan优点：1. 复杂度虽然是O(n)，但是通过游标分布进行的，不会阻塞线程。2. 提供limit参数 3. 同keys一样，也提供模式匹配 4. 服务器你不需要为游标保存状态，游标唯一状态就是为客户端返回游标整数 5. 返回的结果可能会有重复，需要客户端去重 6. 遍历定的过程中如果有数据修改，改动后的数据能不能遍历到是不确定的 7. 单次返回的结果是空的并不意味着遍历结束，而是要看游标值是否为0

### 1.11.1 scan基本用法
- scan提供三个参数，第一个是cursor整数值，第二个是key的正则模式，第三个是遍历的limit hint。
- limit不是返回的数量结果，是单词遍历的字典槽位数量（约等于）

### 1.11.2 字典的结构
- 在Redis里所有的key都存储在一个很大的字典中，类似HashMap，是一维数组，二维联表结构
- scan返回的游标就是第一维数组的位置索引，我们称之为槽

### 1.11.3 scan遍历顺序
高位进位加法来遍历，避免扩容或缩容时槽位的遍历重复和遗漏

### 1.11.4 字典扩容
Java的HashMap扩容，重新分配2倍大小数组，所有元素rehash新的数组下面，rehash相当于元素的hash值对数组长度进行取模运算，因为数组的长度是2的n次方，所以等价于位与操作。7，15，31成为字典的mask值，mask的作用就是保留hash值的低位，高位被置为0

### 1.11.5 对比扩容、缩容前后的遍历顺序
高位进位加法的遍历顺序，rehash后的槽位在遍历顺序上是相邻的。扩容可以避免重复遍历，缩容会有重复遍历

### 1.11.6 渐进式rehash
Java的扩容，会将HashMap一次性rehash，Redis需要使用渐进式rehash，先同时保留旧数组和新数组，定时任务中以及后续对hash指令操作中渐渐地将就数组中挂接的元素迁移到新数组

### 1.11.7 更多的scan指令
zscan遍历zset集合元素，hscan遍历hash字典的元素，sscan遍历set集合的元素

### 1.11.8 大key扫描
- 平时业务逻辑，要尽量避免大key产生。会引起卡顿
- redis-cli -h 127.0.0.1 -p 7001 --bigkeys扫描大key，可以加个休眠参数 -i 0.1

# 第2篇 原理篇

## 2.1 鞭辟入里 — 线程IO模型
Redis是个单线程程序
对于那些O(n)级别的指令，一定要谨慎使用

### 2.1.1 非阻塞IO
非阻塞IO在套接字对象上提供了一个选项Non_Blocking，当这个选项打开时，读写方法不会阻塞，而是能读多少读多少，能写多少写多少。

### 2.1.2 事件轮询（多路复用）
最简单的事件轮询API是select函数，它是操作系统他提供给用户程序的API。输入是读写描述符列表read_fds & writ_fds，输出是与之对应的可读可写时间。同时还提供了一个timeout参数，如果没有任何事件到来，那么就最多等待timeout的值的时间，线程处于阻塞状态。一旦期间有任何事件到来，就可以立即返回。时间过了之后还是没有任何事件到来，也会立即返回。

### 2.1.3 指令队列
Redis会将每个客户端套接字都关联一个指令队列。客户端的指令通过队列来排队进行顺序处理，先到先服务。

### 2.1.4 响应队列
Redis同样也会为每个客户端套接字关联一个响应队列。Redis服务器通过响应队列来将指令的返回结果回复给客户端。

### 2.1.5 定时任务
Redis的定时任务会记录在一个被称为“最小堆”的数据结构中，在这个堆中，最快要执行的任务排在堆的最上方。

## 2.2 交头接耳 — 通信协议
Redis将所有数据都放入内存中，用一个单线程对外提供服务，单个节点在跑满一个CPU核心的情况下可以达到了10W/s的超高QPS

### 2.2.1 RESP
RESP是Redis序列化协议（Redis Serialization Protocol）的缩写，它是一种直观的文本协议，优势在于实现过程异常简单，解析性能极好。
Redis协议将传输分为5种最小单元类型，单元结束时统一加上回车换行符号\r\n
1. 单行字符串以”+”符号开头
2. 多行字符串以“$”符号开头，后跟字符串长度
3. 整数值以“:”符号开头，后跟整数的字符串形式
4. 错误消息以“-”符号开头
5. 数组以”*”号开头，后跟数组的长度

### 2.2.2 客户端 -> 服务器
客户端向服务器发送的指令只有一种格式，多行字符串数组。

### 2.2.3 服务器 -> 客户端
服务器向客户端回复的响应要支持多种数据结构，但再复杂的消息也不会超过以上5种数据结构

### 2.2.4 小结
Redis协议里虽然有大量冗余的回车换行符，但不影响它成为互联网技术领域非常受欢迎的文本协议。在技术领域里，性能并不总是一切，还有简单性、易理解性和易实现性，这些都需要进行适当权衡。

## 2.3 未雨绸缪 — 持久化
- Redis持久化机制有两种，第一种是快照，第二种是AOF日志。快照是一次全量备份，AOF日志是连续的增量备份。
- 快照是内存数据的二进制序列化形式，在存储上非常紧凑，而AOF日志记录的是内存数据修改的指令记录文本
- AOF日志在长期的运行过程中会变得无比庞大，数据库重启时需要加载AOF日志进行指令重放，这个时间就会无比漫长，所以需要定期进行AOF重写，给AOF日志进行瘦身

### 2.3.1 快照原理
- 为了不阻塞线上的业务，Redis就需要一边持久化，一边响应客户端请求。持久化的同事，内存数据结构还在改变
- Redis使用操作系统的多进程COW（Copy On Write）机制来实现快照持久化。

### 2.3.2 fork（多进程）
- Redis在持久化时会调用glibc的函数fork产生一个子进程，快照持久化完全交给子进程来处理，父进程继续处理客户端请求。子进程刚产生的时，它和父进程共享内存里面的代码段和数据段。
- 子进程做数据持久化，不会修改现在的内存数据结构，它只是对数据结构进行遍历读取，然后序列化写道磁盘中。但是父进程不一样，它必须持续服务客户端请求，然后对内存数据结构进行不间断的修改。
- 这个时候就会使用操作系统的COW机制进行数据段页面的分离。数据段是由操作系统的页面组合而成，当父进程对其中一个页面的数据进行修改时，会被共享的页面复制一份分离出来，然后对这个复制的页面进行修改。这时子进程相应的页面是没有变化的，还是进程产生时那一瞬间的数据。

### 2.3.3 AOF原理
Redis会在收到客户端修改指令后，进行参数校验、逻辑处理，如果没问题，就立即将该指令文本存储到AOF日志中，也就说，先执行指令才将日志存盘。不同于leveldb、hbase等存储引擎

### 2.3.4 AOF重写
Redis提供了bgrewriteaof指令用于对AOF日志进行瘦身，其原理就是开辟一个子进程对内存进行遍历，转化成一系列Redis的操作指令，序列化到一个新的AOF日志中。序列化完毕后再将操作期间发生的增量AOF日志追加到这个新的AOF日志文件中，追加完毕后就立即替换旧的AOF日志文件了。

### 2.3.5 fsync
- Linux的glibc提供了fsync(int fd)函数可以将指定文件的内容强制从内核缓存刷到磁盘。只要Redis进程实时调用fsync函数就可以保证AOF日志不丢失。但是fsync是一个磁盘IO操作，它很慢！如果Redis执行一条指令就fsync一次，那么Redis高性能的低位就不保了。
- 所以在生产环境的服务器中，Redis通常是每隔1s左右执行一次fsync操作，这个1s是可以配置的。这是在数据安全性和性能之间做的一个折中，在保持高性能的同时，尽可能使数据少丢失。

### 2.3.6 运维
Redis的主节点不会进行持久化操作，持久化操作主要在从节点进行。从节点是备份节点，没有来自客户端请求的压力，它的操作系统资源往往比较充沛

### 2.3.7 Redis 4.0混合持久化
- 重启Redis时，我们很少使用rdb来回复内存状态，因为会丢失大量数据。我们通常使用AOF日志重放，但是重放AOF日志相对于使用rdb要慢的多。
- 于是在Redis重启的时候，可以先加载rdb的内容，然后再重放增量AOF日志，就可以完全替代之前的AOF全量文件重放，重启效率因此得到大幅提升。

## 2.4 雷厉风行 — 管道
- Redis管道（Pipeline）本身并不是Redis服务器直接提供的技术，这个技术本质上是由客户端提供的，跟服务器没有什么直接关系。

### 2.4.1 Redis的消息交互
客户端经理写-读-写-读四个操作才能完整的执行两条指令，如果我们调整读写顺序，改为写-写-读-读，这两个指令同样可以完成。客户端对管道中的指令列表改变读写顺序就可以大幅节省IO时间。管道中指令越多，效果越好

### 2.4.2 管道压力测试
redis-benchmark，P参数

### 2.4.5 深入理解管道本质
对于管道来说，连续的write操作根本就没有耗时，之后第一个read操作会等待一个网络的来回开销，然后所有的响应消息都已经送回到内核的读缓冲了，后续的read操作直接就可以从缓冲中拿到结果，瞬间就返回了

## 2.5 同舟共济 — 事务

### 2.5.1 Redis事务的基本用法
指令分别是multi、exec、discard。multi表示事务的开始，exec指示事务的执行，discard指示事务的丢弃

### 2.5.2 原子性
Redis的事务根本不具备”原子性“，而仅仅是满足了事务的”隔离性“中的串行化 — 当前执行的事务有着不被其他事务打断的权利

### 2.5.3 discard（丢弃）
在discard之后，队列中的所有制令都没执行

### 2.5.4 优化
通常Redis的客户端在执行事务的都会结合pipeline一起使用，这样可以将多次IO操作压缩为单次IO操作

### 2.5.5 watch
- 两个并发的客户端对账户余额进行修改操作，需要取出余额在内存乘以倍数，将结果写回Redis
- 分布式锁是一种悲观锁
- Redis提供的watch机制，它是一种乐观锁
- watch会再事务开始之前盯住一个或者多个关键变量，当事务执行时，也就是服务器收到了exec指令要顺序执行缓存的事务的队列时，Redis会检查关键变量自watch之后是否被修改了（包括当前事务所在的客户端）。如果关键变量被人东莞过了，exec指令就会返回NULL回复告知客户端事务执行失败，这个时候客户端一般会选择重试

### 2.5.6 注意事项
Redis禁止在multi和exec之间执行watch指令，必须在multi之前盯住关键变量
