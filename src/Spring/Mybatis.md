## 分页插件
实现 Mybatis 提供的接口，实现自定义插件，在插件的拦截方法内拦截待执行的 sql，然后重写 sql。

## Hibernate
Hibernate 对象/关系映射能力强，数据库无关性好，对于关系模型要求高的软件（例如需求固定的定制化软件）如果用 hibernate 开发可以节省很多代码，提高效率。
但是，相比于 Hibernate，我用 Mybatis-Plus，岂不是更香哈哈哈

## 简述 Mybatis 的 Xml 映射文件和 Mybatis 内部数据结构之间的映射关系？
答：Mybatis 将所有 Xml 配置信息都封装到 All-In-One 重量级对象 Configuration 内部。在
Xml 映射文件中，<parameterMap>标签会被解析为 ParameterMap 对象，其每个子元素会
被解析为 ParameterMapping 对象。<resultMap>标签会被解析为 ResultMap 对象，其每个子
元素会被解析为 ResultMapping 对象。每一个<select>、<insert>、<update>、<delete>标签
均会被解析为 MappedStatement 对象，标签内的 sql 会被解析为 BoundSql 对象。

这些都是在启动的时候处理的。starter

### <resultMap>
有了列名与属性名的映射关系后，Mybatis 通过反射创建对象，同时使用反射给对象的属性逐一赋值并返回，那些找不到映射关系的属性，是无法完成赋值的。

## 通常一个 Xml 映射文件，都会写一个 Dao 接口与之对应, Dao 的工作原理，是否可以重载？
答：不能重载，因为通过 Dao 寻找 Xml 对应的 sql 的时候全限名+方法名的保存和寻找策略。接口工作原理为 jdk 动态代理原理，运行时会为 dao 生成 proxy，代理对象会拦截接口方法，去执行对应的 sql 返回数据。