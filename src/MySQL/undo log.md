## undo log

简介undo log、truncate、undo log有哪些？又长啥样？以及undo log如何帮你回滚事务
undolog链条、ReadView、以及undo log如何帮你实现MVCC多版本并发控制

MySQL是单进程多线程的数据库软件，在事务的并发操作中可能会出现脏读，不可重复读，幻读。

因为 Undo Log 可以分为：
Insert Undo Log
Update Undo Log
所以更准确的说法是，Purge 线程清理的对象是 Update Undo Log 和 行记录，因为 Insert Undo Log 会在事务提交之后就会被删除。
我们都知道 InnoDB 的 MVCC 的数据来源是一个一个 Undo Log 形成的单链表，而 Purge 线程就是用于定期清理 Undo Log 的，并且在清理完 删除数据所生成的 Undo Log 的时候，就会把对应的行记录给移除了。

你可以将执行 Purge 操作的线程（简称 Purge 线程）理解成一个后台周期性执行的线程。

fsync系统调用：需要你在入参的位置上传递给他一个fd，然后系统调用就会对这个fd指向的文件起作用。fsync会确保一直到写磁盘操作结束才会返回，所以当你的程序使用这个函数并且它成功返回时，就说明数据肯定已经安全的落盘了。所以fsync适合数据库这种程序。

不要诧异在MySQL专题中突然插入fsync系统调用图片，因为马上就要和大家分享MySQL的undo log、redo log、bin log
为了方便理解，你可以回想一下你擅长使用的编程语言操作文件时，总会贴心的为你提供一个write()方法还有一个flush()方法。
这里的输出方式就是大家耳熟能详的：延迟写
这个缓冲区就是大家耳熟能详的：OS Cache
延迟写降低了磁盘读写的次数，但同时也降低了文件的更新速度。
这样当OS Crash时由于这种延迟写的机制可能会造成文件更新内容的丢失。而为了保证磁盘上的实际文件和缓冲区中的内容保持一致，UNIX系统提供了三个系统调用：sync、fsync、fdatasync

## 长事务的风险

事务迟迟不结束，就意味着它随时可能会访问到数据库中任何数据，所以只要是它们可能用的回滚记录，数据库都得为它们保留着。所以事务越长，相应的他对应的视图也就越大。

上一篇文章中白日梦有和大家介绍过 undo log 默认存放在共享表空间文件中，同在MySQL5.6 MySQL5.7中也允许你将undo log拿到单独的表空间中去，但是不论怎样，undo log总会以真实存在的文件的形式存在于磁盘上，当然了MySQL5.7的undo  truncate机制 结合purge线程可以将不需要的undo log清除掉，为undo log文件瘦身，但是在这个版本之前的MySQL的undo log的体量会不断的增大，再加上大量的长事务，很可能会将磁盘打爆。

另外长事务大概率是update等DML导致的，这种DML是会持有行锁的。谁也不能保证长时间不释放锁不会导致数据库被拖垮。