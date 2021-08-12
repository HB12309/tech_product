因为 Undo Log 可以分为：
Insert Undo Log
Update Undo Log
所以更准确的说法是，Purge 线程清理的对象是 Update Undo Log 和 行记录，因为 Insert Undo Log 会在事务提交之后就会被删除。
我们都知道 InnoDB 的 MVCC 的数据来源是一个一个 Undo Log 形成的单链表，而 Purge 线程就是用于定期清理 Undo Log 的，并且在清理完 删除数据所生成的 Undo Log 的时候，就会把对应的行记录给移除了。

你可以将执行 Purge 操作的线程（简称 Purge 线程）理解成一个后台周期性执行的线程。