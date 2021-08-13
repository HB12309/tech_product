```
DROP TEMPORARY TABLE help_temp;
```

TEMPORARY TABLE &  TABLE 的区别

临时表与普通表的主要区别在于是否在实例，会话，或语句结束后，自动清理数据。比如，内部临时表，我们在一个查询中，如果要存储中间结果集，而查询结束后，临时表就会自动回收，不会影响用户表结构和数据。另外就是，不同会话的临时表可以重名，所有多个会话执行查询时，如果要使用临时表，不会有重名的担忧。5.7引入了临时表空间后，所有临时表都存储在临时表空间(非压缩)中，临时表空间的数据可以复用。临时表并非只支持Innodb引擎，还支持myisam引擎，memory引擎等。因此，临时表我们看不到实体(idb文件)，但其实不一定是内存表，也可能存储在临时表空间中。

理解：就是namespace: session_id下，并且MySQL来维护其生命周期，还有就是存储位置不同。


可以看到，5.7.21 的默认模式包含：

```java
ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```

而第一个：ONLY_FULL_GROUP_BY 就会约束：当我们进行聚合查询的时候，SELECT 的列不能直接包含非 GROUP BY 子句中的列。那如果我们去掉该模式（从“严格模式”到“宽松模式”）呢 ？

### 为什么聚合后不能再引用原表中的列

但需要注意的是，这里的 cname 只是每个学生的属性，并不是小组的属性，而 GROUP BY 又是聚合操作，操作的对象就是由多个学生组成的小组，因此，小组的属性只能是平均或者总和等统计性质的属性.

## order by

将查询所需的字段全部读取到sort_buffer中，就是全字段排序。这里面，有些小伙伴可能会有个疑问,把查询的所有字段都放到sort_buffer，而sort_buffer是一块内存来的，如果数据量太大，sort_buffer放不下怎么办呢？

全字段排序：sort_buffer内存不够的话，就需要用到磁盘临时文件，造成磁盘访问。
rowid排序：sort_buffer可以放更多数据，但是需要再回到原表去取数据，比全字段排序多一次回表。
一般情况下，对于InnoDB存储引擎，会优先使用全字段排序。可以发现 max_length_for_sort_data 参数设置为1024，这个数比较大的。一般情况下，排序字段不会超过这个值，也就是都会走全字段排序。


