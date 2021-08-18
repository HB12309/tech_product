MySQL使用redo log解决了这个问题，redo故名思义：重做。

当发生事务（增、删、改）时会导致缓存页变成脏页，于此同时MySQL会将事务涉及到的：对 XXX表空间中的XXX数据页XXX偏移量的地方做了XXX更新。

所以MySQL意外宕机重启也没关系。只要在重启时解析redo log中的事务然后重放一遍。将Buffer Pool中的缓存页重做成脏页。后续再在合适的时机将该脏页刷入磁盘即可。

这个redo log buffer 中会划分出多个rodo log block。redo log buffer 占用一块连续的内存空间，默认大小16MB。且MySQL允许我们通过参数innodb_log_buffer_size动态的调整它。增大它的大小可以让MySQL处理大事物是不必写入磁盘。进而提升写IO性能。

因为在MySQL的设定中，当你要Commit事务时，redolog才会持久化进磁盘，既然你没有commit，碰巧MySQL又宕机了。那让MySQL正常重启就好了啊，反正你没有commit，MySQL也也没有必要帮你恢复什么。

事务提交时，率先将redo log持久化进磁盘。

innodb将log buffer中的redo log block刷新到上图中的logfile中时，以追加的方式循环写入。也就是首先在ib_logfile0的尾部追加，写满后再写ib_logfile1。当ib_logfile1写满时，清空一部分ib_logfile0接着追加写。

redo log file的大小对innodb性能影响非常大。通常来说要设置的足够大，大到可以让MySQL支持1小时线上高峰流量的接入而不切换。但是设置的过大，数据恢复的时间比较长。设置过小导致循环切换redo log file。

## checkpoint

1、所谓的崩溃恢复，其实就是MySQL重启时照着redo log中的最后一次Checkpoint之后的日志回放一遍

2、因为Checkpoint会不断的更新，并且MySQL重启时只需要对Checkpoint之后的数据进行恢复，所以Checkpoint会缩短MySQL重启的时间。

3、因此每次进行Checkpoint时buffer pool中的脏数据页、redo log中的脏日志都会落盘。所以Checkpoint实际上起到了为这两者进行瘦身的作用。维持两个的可用性。

LSN全称是：log sequence number。

就是一个序列号。并且表空间中的数据页、缓存页、内存中的rodo log、磁盘中的redo log以及checkponit都有LSN标记。

LSN又啥用呢？比如MySQL重启时会对比数据页的LSN和redo log的LSN的大小，如果前者的LSN比后者小。说明数据页中缺失了一部分数据。如果满足其他数据恢复的条件，MySQL就会将LSN之后的这些redo 进行一次回放，完成数据的恢复。