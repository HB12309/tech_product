binlog的高速缓存

意思是：所有未commit的事物产生的binlog，都会被先记录到binlog的高速缓存中。等该事物被commit时，再将缓存中的数据写入binlog日志文件中。

binlog写入磁盘的机制由参数sync_binlog控制。

策略1:sync_binlog = 0

当设置sync_binlog = 0时，表示innodb不会主动控制将binlog落盘，innodb仅仅会将binlog写入到OS Cache中，至于什么时间将binlog刷入磁盘中完全依赖于操作系统。选这种策略，一旦操作系统宕机，OS Cache中的binlog就会丢失。

策略2:sync_binlog = 1

设置sync_binlog = 1时，表示事物commit时将binlog落盘！这样哪怕机器宕机了，也能确保binlog会被写入到磁盘中。

策略3:sync_binlog=N

这里的N不是0，也不是1。

当N大于1时，表示开启组提交，也就是group commit，如果你之前不曾了解组提交的话，你可以这样理解它：比如N=5，那MySQL就会等收集5个binlog后再将这5个binlog一口气同步到磁盘上。好处很明显，一次IO可以往磁盘上刷入N个binlog，IO效率会有所提升。坏处也很明显，比如N=5，那当MySQL收集了4个binlog时，服务器宕机，这4个binlog就会丢失。

一般线上的MySQL的binlog日志格式都是 ROW 格式。

使用 InnoDB 存储引擎，并且事务隔离级别为 READ COMMITTED 或 READ UNCOMMITTED，那binlog的格式只能设置为ROW格式。

一般你公司里面都是在用MySQL集群，集群使用的binlog格式也是ROW格式，你可以捣乱将格式换成 STATEMENT，但会那样报错中断主从复制，而且会导致InnoDB不能再进行插入

## bin log有哪些格式？有啥区别？优缺点？线上用哪种格式？

比如undo log让MySQL有回滚事物的能力，redo log让MySQL有崩溃恢复的能力，以及我们现在说的bin log让MySQL有搭建集群、数据备份、恢复数据的能力。

binlog是由MySQL的Server层产生。
目录下有两种文件：mysql-bin.0000XX 和 mysql-bin.index

比如你线上使用的是一个一主两从的MySQL集群，然后有一个活动来了，你需要给线上某库的某数据表添加一列，并且这个表里面的数量非常之庞大。

添加一列是需要获取表锁的，并且庞大的数据量让你alter table异常缓慢，在获取表锁的过程中，正常的DML也会被阻塞。这时你就得考虑无损DDL，比如golang的ghost（怎么做的无损ddl原理我后门的文章会写的，本篇不展开）。

ghost工具的特点是，它需要预执行一些SQL目的是先校验一下你的集群符不符合它的要求，为了不对线上产生影响，你肯定会想把这些预执行的sql放到从库上执行。放在从库上执行固然没问题，但是你执行的sql会产生bin log。出现新的GTID（后面讲MySQL集群时会跟大家分享，这里你只需要理解成是一个事物的唯一标识就行）更要命的是，主库中没有这些GTID。

当主从都正常运行时，主从bin log不一致没关系，但是当从库宕机的时候，从库重启会将自己的GTID集合发送给主库（对比GTID可以知道该给从库哪些binlog的数据），由于从库多出来一部分主库没有的GTID，会导致该从库不能再次加入集群。

其实这个问题也好解决。使用这个参数 sql_log_bin 

该sql_log_bin变量控制是否为当前会话启用到二进制日志的日志记录（假设二进制日志本身已启用）。默认值为ON。要为当前会话禁用或启用二进制日志记录，请将会话sql_log_bin变量设置为 OFF或ON。

全局sql_log_bin变量是只读的，无法修改。

## 备份

MySQL中数据备份：常见的有冷备份、逻辑备份、热备份、快照备份。

冷备份：数据库停止运行的情况下，直接备份磁盘中MySQL用来存储数据的那些数据文件。进入到如下的目录中，你可以看到MySQL为我们创建的数据库表创建出了单独的目录，而目录中的有 .frm、.idb文件就是冷备份需要备份的文件。

逻辑备份：

为啥说mysqldump是逻辑备份？原因大概是：你使用mysqldump去备份最终得到的参数其实是一堆sql，再通过回放sql的形式完成数据的恢复。白日梦之前的文章中跟大家分享过（可自行查看历史文章哈）。在MySQL中数据表、数据行其实是逻辑存上的概念。像数据页这种概念是物理真实存在的。所以你用mysqldump得到一堆sql，自然称得上是逻辑备份喽。

热备份：

xtraback 的工具完成数据库的热备份。
除此之外，我了解有一款Golang写的开源工具 ghost，在github上还是挺火的。它是一款支持做无损DDL的工具（后面会专门有一篇文章讲这个工具的原理）。这款工具在实现支持无损DDL功能时，有一部分逻辑本质上也是在支持增量数据的备份。
ghost的实现手段是：添加binlog监听事件，监听到binlog event后去解析binlog得到sql，再回放这个SQL。就像是从库使用主库对binlog进行数据恢复一样。

快照备份：
快照备份不是数据库本身提供的能力，本质上它是借助于文件系统的快照功能来实现的对数据库的备份。
我们知道的Linux服务器本质上也是电脑的，它会有自己的磁盘，无论是固态硬盘，还是机械磁盘。反正会有这种固态存储。还需要进一步对磁盘进行分区。然后才有将Linux文件系统中的目录都会挂载在不同的分区上。这么做的目的，简单来说就像你的window有C盘、D盘、E盘。D盘中的出问题后不会影响E盘一样。
快照备份要求：数据库的所有数据文件都要放在一个数据分区中。

比如一条update有语句进入MySQL之后经历如下过程：

```
# 这也是所谓的两阶段提交

1. 写undolog # 回滚
2. 写redolog（prepare）# 保证提交的不会丢失
3. 写一个特殊的Binlog Event，类型为GTID_Event，指定下一个事务的GTID 
4. 写binlog # 主从同步事物使用
5. 写redolog（commit）

```

而在这种方式出现之前，主从之间同步数据时，从库需要告诉主库自己已经同步到binlog.0000x，position=yyy的地方了。这个binlog.0000x，position=yyy需要人为的去查看一下。不能说查看这两个信息比较麻烦，但是肯定不如GTID来的方便。

线上的数据库不断承接流量，binlog会不断滚动变大，你要赶在binlog被清理之前去恢复数据。

假设你没有赶在binlog被清理之前去恢复数据，当你去恢复数据时上图中delete sql之前的binlog已经被删除了。那怎么办？
这时你可以通过最近的全量备份把delete之前的数据恢复出来，然后delete之后的增量数据，通过mysqlbinlog工具恢复出来，注意别忘了通过positon跳过这个delete，不然一执行会放出来delete语句，数据又全被删除了。
如果你没有全量备份，binlog也不全了。那估计就悬了！

## 两阶段提交

明明没有显示的添加begin、commit命令，但是MySQL实际执行我的SQL时，竟然为我添加上了。一般大家的线上库都会将这个参数置为ON，你的SQL会自动的开启一个事物，并且MySQL会自动的帮你把它提交。

binlog默认都是不开启的状态！
也就是说，如果你根本不需要binlog带给你的特性（比如数据备份恢复、搭建MySQL主从集群），那你根本就用不着让MySQL写binlog，也用不着什么两阶段提交。
只用一个redolog就够了。无论你的数据库如何crash，redolog中记录的内容总能让你MySQL内存中的数据恢复成crash之前的状态。
所以说，两阶段提交的主要用意是：为了保证redolog和binlog数据的安全一致性。只有在这两个日志文件逻辑上高度一致了。你才能放心的使用redolog帮你将数据库中的状态恢复成crash之前的状态，使用binlog实现数据备份、恢复、以及主从复制。而两阶段提交的机制可以保证这两个日志文件的逻辑是高度一致的。没有错误、没有冲突。

假如要执行一条update语句，那你肯定知道，先写undolog（便于后续对update事务的回滚）。然后你的update逻辑将Buffer Pool中的缓存页修改成了脏页。
当你准备提交事物时（也就是step1阶段），会写redolog，并将其标记为prepare阶段。然后再写binlog，并将binlog落盘。
这时发生了意外，MySQL宕机了。
那我问你，当你重启MySQL后，update对BufferPool中做出的修改是会被回滚还是会被提交呢？
答案是：会根据redolog将修改后的recovey出来，然后提交。
那为什么会这样做呢？
其实总的来说，不论mysql什么时刻crash，最终是commit还是rollback完全取决于MySQL能不能判断出binlog和redolog在逻辑上是否达成了一致。只要逻辑上达成了一致就可以commit，否则只能rollback。
比如还是上面描述的场景，binlog已经写了，如果MySQL最终选择了回滚。那代表你的binlog比BufferPool（或者Disk）中的真实数据多出一条更新，日后你用这份binlog做数据恢复，是不是结果一定是错误的？
或者说binlog已经落盘了，那很有可能所有的从库都把这条binlog拿走自己回放了，这时如果主库选择回滚丢弃这条数据，那是不是主从数据不一致了？

当MySQL写完redolog并将它标记为prepare状态时，并且会在redolog中记录一个XID，它全局唯一的标识着这个事务。而当你设置`sync_binlog=1`时，做完了上面第一阶段写redolog后，mysql就会对应binlog并且会直接将其刷新到磁盘中。
下图就是磁盘上的row格式的binlog记录。binlog结束的位置上也有一个XID。
只要这个XID和redolog中记录的XID是一致的，MySQL就会认为binlog和redolog逻辑上一致。就上面的场景来说就会commit，而如果仅仅是rodolog中记录了XID，binlog中没有，MySQL就会RollBack


CREATE TABLE `kemu` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT 'id',
  `kemu` varchar(50)  NOT NULL DEFAULT '' COMMENT 'kemu',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='kemu';

CREATE TABLE `score` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT 'id',
  `score` int(5)  NOT NULL DEFAULT 0 COMMENT 'score',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='score';

insert kemu (id, kemu) values 
    (1,'语文'),(2,'数学'),(3,'英语')
;

insert score (id, score) values 
    (2,70),(3,80),(4,90)
;