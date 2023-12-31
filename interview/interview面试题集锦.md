一定要准备好以下问题：

@[TOC]

# 一、综合篇

## 1、自我介绍

**重在突出自己对当前面试岗位的匹配度。**【面试就是在十几分钟内跟面试官推销自己的过程】

- 2020年硕士毕业于南京邮电大学，之后一直在中国移动软件技术有限公司工作。
- 因为个人家庭原因，目前在找一份工作地点位于上海的工作。最近看到贵公司在招GO开发工程师，本人也有意转到这个方向。希望日后有机会可以从事该方向的工作。
- 我的工作：目前是在做部门的核心产品——数据库产品的业务开发，核心职责是负责组内MySQL产品的订退改续以及容量管理工作，主要目的就是保证客户的订单成功率。并将经验推广到部门的其它数据库类产品，当年工作绩效也因此比较好，是同批校招中第一个竞聘升岗的。
- 最近是在做统一的订购服务模块，将每个产品都需要对接的订退改续部分抽成一个公共模块，这样不仅借由核心产品MySQL的订单成功率来提高其它产品的成功率，还节约了大量的人力成本。
- 技术栈：Go/java + MySQL/Redis/MongoDB + Kafka/RocketMQ
- 我也积极参与分享，在CSDN上以博客方式分享自己的工作与学习经验，阅读量超20W+。
- 近期关注到贵公司在招聘 XXX，我认为我和这个岗位比较契合，相比其它候选人，我认为我的优势在于 XX。




- 面试自我介绍模板
- 见目录：interview/interview文件存档/自我介绍模板.png

## 2、项目介绍。项目上有哪些是自己做的比较好的设计或者优化？

- 项目介绍思路：
  - 1、项目立项时所处的背景
  - 2、项目立项时业务所面临的困境
  - 3、项目实施时所用到的综合技术方案
  - 4、项目上线后所带来的的价值

- 进销存系统

- 统一网关emop-gateway系统是我们组的核心系统，它主要解决对接第三方平台，如移动云平台【op/mop】、计费模块【EBOSS】、运维平台【OPS】等第三方平台。
  它的核心难点是要保证高可用下的数据一致性。 为此我将整个模块做成了一个对外的网关，为业务方屏蔽了第三方平台的差异。并且在这基础上，
  加入了统一的服务治理、容错、鉴权等措施，可用性提高到了 四个9，和原本比起来出错率下降了90%。
  同时我还设计了一个支持测试的mock服务，使得开发和测试效率提高了20%。因此当年绩效比较好，是同批校招中第一个竞聘升岗。

- 认证模块

- 统一订购模块服务

## 3、项目中核心困难？怎么解决的？

- 回答思路：

  - 发现问题
  - 分析问题
  - 提出方案
  - 取得效果。

- 案例一、进销存算法——用户订购的规格落到k8s底层的模型建立过程。
  - todo

- 案例二、Order访问元数据库超时问题排查——order连接mysql异常：i/o timeout 或者 invalid connection 报错。
  - 1、定时备份导致IO使用率高，引发连接异常。general-log定时归档导致IO使用率高。
  - 2、mysql所在宿主机内核软死锁(soft lockup) 导致主从切换引发的连接异常，该问题可能是由于虚机所在的宿主机的CPU负载较高或磁盘IO太高所致，
    虚拟机层面无法避免。迁移到物理机。
  - 3、代码层面：代码层面使用gorm连接池，但是未配置相关参数，导致每次查询基本都会建立新连接，频繁的创建新连接，可能会导致连接异常。
  ```shell
  DB.DB().SetConnMaxLifetime(600)    //连接最大存活时长 【数据库连接达到600s后关闭，reuse重用前判断是否超过600s，如果超过，则重建】
  DB.DB().SetConnMaxIdleTime(300)    //连接空闲超时时长 【闲时连接达到300s也关闭】
  DB.DB().SetMaxOpenConns(300)       //连接池最大连接数
  DB.DB().SetMaxIdleConns(20)        //连接池最大空闲连接数
  ```

## 4、工作过程中遇到过哪些问题？最大的挑战是什么？

- RwMutex死锁问题排查：https://blog.csdn.net/qq_41822345/article/details/125907670
- 批量插入数据优化：https://mp.weixin.qq.com/s/-r8NxSuTprK2eS5vyjn31A
- 案例、杭研慢sql问题排查 
  - 背景： 客户业务侧会偶发出现慢SQL日志输出，一周大概100条慢SQL左右。 
  - 客户业务侧通过配置了Druid的慢SQL功能记录相关日志，如下是客户在平台上抓取慢SQL日志后的展示： 
  - 日志中包含了业务节点IP，容器IP，SQL执行时长，详细SQL和对应时间点。 
  - 根据客户提供的慢SQL信息，去所有后端MySQL节点上排查是否存在对应的慢日志：
    - 1、针对业务侧凌晨1-2点批量出现的慢SQL，可以在后端MySQL节点上找到对应的慢日志记录。 
    - 2、针对业务侧其他时间点出现的慢SQL，未在后端MySQL节点上找到对应的慢日志记录。 
    - 针对情况1：因为MySQL中本身也记录了慢SQL，所以肯定是由于性能引发的问题，根据排查，发现凌晨1-2点，MySQL实例节点会出现IOPS飙高的问题，
 此时读写流量激增，会影响到SQL执行效率。通过后台运行监控脚本，确认IOPS飙高是因为其他备库实例备份导致，通过限制备份IOPS后，此类问题基本解决，
 凌晨1-2点间不会因为备份而产生慢SQL。 
    - 针对情况2：猜测不是MySQL自身的问题，因为慢日志中并没有发现对应的慢SQL。后在sre运维人员的配合下通过全链路抓包发现：从全链路抓包分析慢SQL来看，
 耗时主要是在客户通过云主机自建的K8S内部，怀疑是经过客户自建K8S中网络转发有问题，导致SQL从云主机发包已经慢了5秒，进而引发慢SQL。

## 5、工作过程中用到过哪些设计模式？

见目录：架构/设计模式


- 观察者模式：邮件、短信、电话。根据事件等级


## 6、为什么选择来我们公司？

- 上海【todo，对每个城市的看法】：
- 南京：
- 合肥：

- 公司：
- 个人：
- 家庭：

## 7、你的优点？你的缺点？

对我来说，我个是比较着急的人，急性子。所以，我最大的问题就是在推进一些事的时候，会忽略别人的感受。当压力变大的时候，我甚至会说出一些别人
难以接受的话（俗话说的情商为零）。这个没什么不好意思承认的，我这么多年来也在改进自己。

## 8、对当前岗位的看法？你有什么优势？

个人优势：公有云、厂商管理经验。
2022年获得了计算机技术与软件专业资格证书-软件设计师中级证书。

# 二、Go语言

## 1、Java和Go语言对比，各有什么优缺点，各自适合做什么样的工作？
- 1、Java是面向对象的语言，它的基本思想是使用对象、类、封装、继承、多态等基本概念来进行程序设计。Go里只有一个结构体，结构体之间通过组合的方式来复用字段属性或者方法。
- 2、编译：Java先由JVM编译成字节码，再编译成机器码。而Go是直接编译为二进制文件。所有一般来说Go的性能和效率高于Java。
> Java 是 JIT， Just-in-time,动态(即时)编译，边运行边编译；Go是AOT编译，AOT，Ahead Of Time，指运行前编译，预先编译。
- 3、并发，Go语言在设计之初就考虑到了多核处理器，它提供了goroutine的轻量级线程，比Java的Thread更轻量，并发性能也更高些。
- 4、生态：Java流行了20多年，它具有很广泛的生态。Go是后起之秀，基于云原生发展起来。
- 5、异常处理：Java try-catch-finally；Go 一大堆err判断。defer结合panic/recover处理。


## 2、Java和Go在多线程编程上有什么区别？

- Java 中 CPU 资源分配对象是 Thread，Go 中 CPU 资源分配对象是 goroutine。Java Thread 与系统线程为一一对应关系，goroutine 是 Go 实现的用户级线程，与系统线程是 m:n 关系。
- Golang语言的goroutine是一种轻量级的线程，它们的创建和销毁速度比Java中的线程快得多。在Java中，创建和销毁线程都需要相当大的开销。
- Golang语言采用了CSP的并发模型，它通过在并发实体之间进行消息传递来实现并发控制，其中以goroutine和channel作为主要实现手段。 Java则采用了多线程并发模型，其中以Thread和Synchronization作为主要实现手段。
> Do not communicate by sharing memory; instead, share memory by communicating.

## 3、GMP介绍下？



## 4、GO的垃圾回收了解吗？



# 三、Java语言



# 四、MySQL

## 1、有没有MySQL部署调优经验？ 

MySQL调优场景描述【从部署到上线】

- 可以反问几个问题来确认需求：单主多从、16C64G1TB固态盘 
- 答题思路：io线程+bufferpool+表空间文件+同步配置+binlog开启+表结构合理化+压测提高缓冲命中+binlog cache_size调整】
- 1、首先，第一步在MySQL的安装部署阶段，进行参数调优【DBA运维】：
  
    - 参数1：IO线程[分为read_io和write_io线程]【MySQL后台有四个线程：Matser线程、IO线程、purge线程、page_cleaner线程】默认为4个，将其调到6个尽最大可能利用多核CPU；
    - 参数2：innodb_buffer_pool_size默认128M，预先调整为64G的25%，后面可以将其根据业务量的大小调整到64G的75%；
    - 参数3：innodb_buffer_pool_instance默认为1。可以调整为2或者4来作为缓冲池的个数，可以达到一个负载均衡的效果。
    - 参数4：表空间参数，innodb_file_per_table，默认关闭。应该打开，避免单个文件越大。索引越慢，也能达到物理文件的一个负载均衡的效果。
    - 参数5：innodb_data_path开启多个目录。开启表空间文件参数后，除了数据、数据的索引、插入缓冲数据每个表各占用一个文件，其它的数据比如：回滚段数据、事务数据等还是在共享表空间里，开启innodb_data_path的多个目录可以将这些数据负载到多个目录下。
    - 参数6：对于单主多从，binlog默认关闭，需要开启主的binlog。另外将事务隔离级别设置为rc，基于rbr模式进行主从复制，将lock_mod设置为2，以互斥量形式进行id的分发，提高插入性能。
    - 参数7：双一半同步设置：redolog的刷盘机制参数： innodb_flush_log_at_trx_commit为1——提交事务的时候将 redo 日志写入磁盘中（设置为1表示提交事务的时候，就必须把 redo log 从内存刷入到磁盘文件里去）；binlog的刷盘机制参数：sync_binlog —— 用来控制数据库的binlog刷到磁盘上（设置为1表示每次事务提交，MySQL都会把binlog刷下去，是最安全但是性能损耗最大的设置）
- 参数8：开启线程池参数one-thread-per-connection。 开启线程池可以很大程度上能够避免因无法预测的热点数据导致宕机的事件发生。开启了1024个客户端，同时发送sql请求，mysql实例的qps也能稳定在最高值12000左右，相比于原来未开启线程池时的qps在1200左右，提高了9倍。 阿里云rds版本更新中有说明，本次更新数据库实例qps提高了数十倍，实际上就是开启了线程池，在多客户端请求下，qps稳定在最大值。
  
- 2、其次，第二步在业务开发阶段，数据库表设计【程序员】： 
  - 比如严格控制每个字段的大小，尽可能保证B+树的高闪出性；根据业务需求进行索引设计； 
  - 根据二级索引的设计测评是否需要开启5.6或者5.7的mrr优化，调整read_rnd_buffer_size。 
  - 【Multi-Range Read，即多范围读取，主要解决的是当二级索引取出索引值后再去聚集索引中取行可能会造成大量的磁盘随机IO的问题，通过在内存缓冲区的一次排序，将随机I/O变为顺序I/O，提高】

- 3、最后，第三步是在压测阶段：
  - 开启慢查询日志slow_query_log【生产环境需要关闭】，根据具体的业务情况设置long_query_time，超过 long_query_time的sql会被记录下来。 
  - 用explain分析这些耗时的sql语句，从而针对性优化。 
  - 另外，压测过程中，通过执行show global status可以查看innodb缓冲池的命中率： 缓冲池的读取次数/磁盘的读取次数，如果该值小于99%【最优值】，则需要适当的调整缓冲池innodb_buffer_pool_size的大小。

## 2、有没有SQL查询优化经验？如何调优的？调优思路是？业务出现劣化/腐化情况下，这时候是如何进行sql优化的？

- 参考链接：https://mp.weixin.qq.com/s/e0WtvBf-LJkCG4NTxiiStg

- 1、开启慢查询日志，打开`slow_query_log`参数，设置`long_query_time`，记录超过该long_query_time的sql语句。

- 2、explain sql语句，关注 type、rows、filtered、extra、key等字段；
    - type：表示连接类型：system>const>eq_ref>ref>index_merge>unique_subquery>index>all等等
    - rows：表示读取的记录数，估值。
    - filtered：过滤后剩下的记录数据的比率，越小越好。
    - extra：表示MySQL解析查询的其它信息，比如：
       - use fileSort：文件排序。一般出现在order by语句。
       - use index：表示使用覆盖索引。
       - use where：表示使用where条件过滤。
       - use temporary：表示使用临时表。性能差，需要格外优化。一般出现在group by语句。
       - use index condition：表示使用索引下推。利用索引在存储引擎层进行数据过滤，以减少回表。
    - key：查看是否命中了索引。
```shell
mysql> explain SELECT * FROM `OrdersTable`  WHERE `OrdersTable`.`deleted_at` IS NULL AND ((parent_uuid = 'c7240f4c4dc4a20155a10' and order_type = 1 ));
+----+-------------+-------------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+------------------------------------+
| id | select_type |   table     | partitions | type | possible_keys         | key                   | key_len | ref   | rows | filtered | Extra                              |
+----+-------------+-------------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+------------------------------------+
|  1 | SIMPLE      | OrdersTable | NULL       | ref  | idx_Orders_deleted_at | idx_Orders_deleted_at | 5       | const | 1419 |     1.00 | Using index condition; Using where |
+----+-------------+-------------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+------------------------------------+
1 row in set, 1 warning (0.00 sec)
```
    
- 3、profile分析，explain是预估，profile可以进一步了解SQL执行的状态和时间消耗。需要开启`profiling`参数。打开后，执行完SQL语句后，再执行`show profile`或者`show profile for query_id`查看。 

- 4、Optimizer Trace可以跟踪执行语句的解析优化全过程。开启`set optimizer_trace="enable=on"`，然后执行sql，执行完sql之后查看表`select * from information_schema.optimizer_trace`，查看其执行树，一般分为三个阶段：join_preparation、join_optimization、join_execution。

- 5、最后确认问题，采取对应的措施。
    - 多数慢SQL都跟索引有关，比如不加索引，索引失效、不合理等，这时候可以优化索引。
    - 还可以优化SQL语句，见下一个面试题。
    - 如果单表数据量过大导致慢查询，则可以考虑分库分表。
    - 如果存量数据量太大，考虑是否可以让部分数据归档。
    - 如果数据库在刷脏页导致慢查询，考虑是否可以优化一些参数，跟DBA讨论优化方案。

## 3、有哪些常见的慢sql场景？
- 1.隐式转换
- 2.最左匹配
- 3.Limit深分页问题
- 4.in子查询元素过多
- 5.Order by走文件排序
- 6.索引字段使用!=或者<>
- 7.左右连接，关联的字段编码格式不一样
- 8.group by使用临时表
- 9.delete+in子查询不走索引

## 4、有哪些MySQL语句性能优化技巧？
参考链接：https://zhuanlan.zhihu.com/p/609481907

## 5、说一说MySQL的B+树索引？

- 它是帮助MySQL高效获取数据的数据结构，主要是用来提高数据检索的效率，降低数据库的IO成本，同时通过索引列对数据进行排序，降低数据排序的成本，也能降低了CPU的消耗。
- 采用的B+树的数据结构来存储索 引，选择B+树的主要原因是【这也是和B树的区别】：
  - 第一是磁盘读写代价更低，非叶子节点只存储指针，叶子节点存储数据。
  - 第二是便于扫库和区间查询，叶子节点是一个双向链表。

## 6、MySQL中有哪些锁？它们有哪些优缺点？
参考链接：https://zhuanlan.zhihu.com/p/636972366

## 7、MySQL在什么场景下会加间隙锁？
参考链接：https://zhuanlan.zhihu.com/p/636972366

## 8、MySQL死锁问题？如何查看？如何解决？
```shell
#查看死锁：
show engine innodb status\G
#解除死锁方法：
#方法一：
show OPEN TABLES where in_use > 0;
show processlist;
#3.kill id;
#方法二：
SELECT * FROM INFORMATION_SCHEMA.INNODB_TRX;
SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCKS;
SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS;
kill PID;
```

## 9、MySQL如何实现原子性、一致性、隔离性和持久性的？
- 持久性：redo log【Buffer Pool缓存中修改的数据如何保证一定刷盘成功→WAL预写日志，所有修改先写入redo log日志，再更新Buffer Pool】
- 原子性：undo log【事务执行失败，则利用undo log中的信息将数据回滚】
- 隔离性：锁和MVCC
  - （一个事务）写操作对（另外一个事务）写操作的影响：锁机制保证隔离性
  - （一个事务）写操作对（另外一个事务）读操作的影响：MVCC机制保证隔离性
  - MVCC实现：隐藏字段、版本链[undo log日志链]、readView
    - 隐藏字段：db_trx_id、db_roll_pointer[版本链的关键]、db_row_id
    - MVCC——https://mp.weixin.qq.com/s/6lQMW6L6w9rGNaggk_rfLQ
- 一致性：前面三个特性保证了一致性。


# 五、Redis

## 1、Redis在工作过程中的使用场景？缓存结构是怎么设计的？Why？缓存的数据结构？数据量？



## 2、使用缓存的实际过程？如何解决缓存一致性问题？



## 3、Redis的主从同步过程？主从过程中延迟遇到的问题？



## 4、Redis的缓存穿透、缓存击穿、缓存雪崩问题？
参考链接：https://blog.csdn.net/qq_41822345/article/details/125568012




# 六、k8s



# 七、Kafka



# 八、RocketMQ





# 九、MongoDB





# 十、算法

常考算法题：https://blog.csdn.net/qq_41822345/article/details/123122339

# 十一、计算机网络

- 参考链接：https://mp.weixin.qq.com/s/QOcxNmjTJPHJsxEay-oVOg

## 1、三次握手？
参考链接：https://blog.csdn.net/qq_41822345/article/details/128882826

## 2、HTTP1.0 和 HTTP1.1和 HTTP2.0 和 HTTP3.0有哪些区别与改进？

参考链接：https://blog.csdn.net/weixin_53186633/article/details/123624445

## 3、SSL协议？

参考链接：https://baike.baidu.com/item/%E5%AE%89%E5%85%A8%E5%A5%97%E6%8E%A5%E5%B1%82/9442234?fr=ge_ala



