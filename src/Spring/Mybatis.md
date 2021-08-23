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

在 MyBatis-Plus 中 Mapper 重载并不会出现异常，但是查询结果都是相同的。因为 MyBatis-Plus 的 MybatisConfiguration 继承重写了 MyBatis Configuration 的 addMappedStatement 方法。
在 MyBatis-Plus 中发现该 MappedStatement 已经存在，则不进行添加。
而在 MyBatis 中如果 MappedStatement 如果 key 存在，则直接抛出异常，服务启动失败。
写了个重载。虽然记得不能重载，但是看启动没问题，就觉得 ok。

## @Scheduled & @EnableScheduling

ScheduledAnnotationBeanPostProcessor是一个bean后置处理器，内部有个postProcessAfterInitialization方法，spring中任何bean在初始化完毕之后，会自动调用postProcessAfterInitialization方法，而ScheduledAnnotationBeanPostProcessor在这个方法中会解析bean中标注有@Scheduled注解的方法，这些方法也就是需要定时执行的方法。
ScheduledAnnotationBeanPostProcessor还实现了一个接口：SmartInitializingSingleton，SmartInitializingSingleton中有个方法afterSingletonsInstantiated会在spring容器中所有单例bean初始化完毕之后调用，定期器的装配及启动都是在这个方法中进行的

### @EnableCaching

spring中的缓存主要是利用spring中aop实现的，通过Aop对需要使用缓存的bean创建代理对象，通过代理对象拦截目标方法的执行，实现缓存功能。
重点在于@EnableCaching这个注解，可以从@Import这个注解看起
最终会给需要使用缓存的bean创建代理对象，并且会在代理中添加一个拦截器org.springframework.cache.interceptor.CacheInterceptor，这个类中的invoke方法是关键，会拦截所有缓存相关的目标方法的执行，大家可以去细看一下。

文中的案例都是以本地内存作为存储介质的，但是实际上我们的项目上线之后，基本上都会采用集群的方式进行部署，如果将数据存储在本地内存中，集群之间是无法共享的，我们可以将数据存储在redis中，从而实现缓存的共享，下面我们一起来看下Spring中@EnableCaching如何对接redis。

JdbcTemplate是Spring对JDBC的封装，目的是使JDBC更加易于使用。
拿来共同比较的jar spring-jdbc mybatis-jdbc

```java
@Configuration
@ComponentScan
public class MainConfig {
    public static void main(String[] args) {
        //1.创建spring上下文
        AnnotationConfigApplicationContext configApplicationContext = new AnnotationConfigApplicationContext();
        //2.上下文中注册bean
        configApplicationContext.register(MainConfig.class);
        //3.刷新spring上下文，内部会启动spring上下文
        configApplicationContext.refresh();
        //4.关闭spring上下文
        System.out.println("stop ok!");
    }
}
```

