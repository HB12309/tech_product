https://www.cnblogs.com/ZhangZiSheng001/p/12329937.html

# 从池中借出的连接是否默认自动提交事务
# 默认 true
autoCommit=true

# 当我从池中借出连接时，愿意等待多长时间。如果超时，将抛出 SQLException
# 默认 30000 ms，最小值 250 ms。支持 JMX 动态修改
connectionTimeout=30000

# 一个连接在池里闲置多久时会被抛弃
# 当 minimumIdle < maximumPoolSize 才生效
# 默认值 600000 ms，最小值为 10000 ms，0表示禁用该功能。支持 JMX 动态修改
idleTimeout=600000

# 多久检查一次连接的活性
# 检查时会先把连接从池中拿出来（空闲的话），然后调用isValid()或执行connectionTestQuery来校验活性，如果通过校验，则放回池里。
# 默认 0 （不启用），最小值为 30000 ms，必须小于 maxLifetime。支持 JMX 动态修改
keepaliveTime=0

# 当一个连接存活了足够久，HikariCP 将会在它空闲时把它抛弃
# 默认 1800000  ms，最小值为 30000 ms，0 表示禁用该功能。支持 JMX 动态修改
maxLifetime=1800000

# 用来检查连接活性的 sql，要求是一个查询语句，常用select 'x'
# 如果驱动支持 JDBC4.0，建议不设置，这时默认会调用  Connection.isValid() 来检查，该方式会更高效一些
# 默认为空
# connectionTestQuery=

# 池中至少要有多少空闲连接。
# 当空闲连接 < minimumIdle，总连接 < maximumPoolSize 时，将新增连接
# 默认等于 maximumPoolSize。支持 JMX 动态修改
minimumIdle=5

# 池中最多容纳多少连接（包括空闲的和在用的）
# 默认为 10。支持 JMX 动态修改
maximumPoolSize=10

# 用于记录连接池各项指标的 MetricRegistry 实现类
# 默认为空，只能通过代码设置
# metricRegistry=

# 用于报告连接池健康状态的 HealthCheckRegistry 实现类
# 默认为空，只能通过代码设置
# healthCheckRegistry=

# 连接池名称。
# 默认自动生成
poolName=zzsCP

# 如果启动连接池时不能成功初始化连接，是否快速失败 TODO
# >0 时，会尝试获取连接。如果获取时间超过指定时长，不会开启连接池，并抛出异常
# =0 时，会尝试获取并验证连接。如果获取成功但验证失败则不开启池，但是如果获取失败还是会开启池
# <0 时，不管是否获取或校验成功都会开启池。
# 默认为 1
initializationFailTimeout=1

# 是否在事务中隔离 HikariCP 自己的查询。
# autoCommit 为 false 时才生效
# 默认 false
isolateInternalQueries=false

# 是否允许通过 JMX 挂起和恢复连接池
# 默认为 false
allowPoolSuspension=false

# 当连接从池中取出时是否设置为只读
# 默认值 false
readOnly=false

# 是否开启 JMX
# 默认 false
registerMbeans=true

# 数据库 catalog
# 默认由驱动决定
# catalog=

# 在每个连接创建后、放入池前，需要执行的初始化语句
# 如果执行失败，该连接会被丢弃
# 默认为空
# connectionInitSql=

# JDBC 驱动使用的 Driver 实现类
# 一般根据 jdbcUrl 判断就行，报错说找不到驱动时才需要加
# 默认为空
# driverClassName=

# 连接的默认事务隔离级别
# 默认值为空，由驱动决定
# transactionIsolation=

# 校验连接活性允许的超时时间
# 默认 5000 ms，最小值为 250 ms，要求小于 connectionTimeout。支持 JMX 动态修改
validationTimeout=5000

# 连接对象可以被借出多久
# 默认 0（不开启），最小允许值为 2000 ms。支持 JMX 动态修改
leakDetectionThreshold=0

# 直接指定 DataSource 实例，而不是通过 dataSourceClassName 来反射构造
# 默认为空，只能通过代码设置
# dataSource=

# 数据库 schema
# 默认由驱动决定
# schema=

# 指定连接池获取线程的 ThreadFactory 实例
# 默认为空，只能通过代码设置
# threadFactory=

# 指定连接池开启定时任务的 ScheduledExecutorService 实例（建议设置setRemoveOnCancelPolicy(true)）
# 默认为空，只能通过代码设置
# scheduledExecutor=

# JNDI 配置的数据源名
# 默认为空
# dataSourceJndiName=

在“传统模型”中，borrow、return、add、remove 四个动作都需要加同一把锁，即同一时刻只允许一个线程操作池，并发高时线程切换将非常频繁。因为多个线程操作同一个池塘，连接出入池需要加锁来保证线程安全。针对这一点，我们是不是能做些什么呢？

首先，我要 borrow 时，我需要看看池塘里哪一个连接可以借出。这里就涉及到读连接池的操作，因为池塘里的连接数量不是一成不变的，为了一致性，我们就必须加锁。但是，HikariCP 没有加，为什么呢？因为 HikariCP 容忍了读的不一致。borrow 的时候，我们实际上读的不是真正的池塘，而是当前池塘的一份快照。我们看看 HikariCP 存放连接的地方，是一个CopyOnWriteArrayList对象，我们知道，CopyOnWriteArrayList是一个写安全、读不安全的集合

于是，在“标记模型”里，只有 add 和 remove 才需要加锁，borrow 和 return 不需要加锁。通过这种颠覆式的设计，连接池的性能得到极大的提高。