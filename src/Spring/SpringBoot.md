spring springmvc  springboot  springcloud，其实考点和问的问题很多。侧重点各不相同，而且从左往右是增量，从右往左就越来越底层。

spring: IOC AOP DI 三级缓存。基础底层，不需要考虑web服务器
springmvc:  DispaterServlet  tomcat 的情况，作为web服务器的逻辑。
springboot: 自动装配，starter逻辑
springcloud: 各式各样的组件

我是在看下面的代码的时候明白的，SpringBoot就是在 @SpringBootApplication 和 starter 下，通过AnnotationApplicationContext等来创建容器，设置，启动的。我们也可以通过xml的方式来手动管理，设置 Scope等等。

```
public class ThreadScopeTest {
    public static void main(String[] args) throws InterruptedException {
        String beanXml = "classpath:/com/javacode2018/lesson001/demo4/beans-thread.xml";
        //手动创建容器
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext();
        //设置配置文件位置
        context.setConfigLocation(beanXml);
        //启动容器
        context.refresh();
        //向容器中注册自定义的scope
        context.getBeanFactory().registerScope(ThreadScope.THREAD_SCOPE, new ThreadScope());//@1

        //使用容器获取bean
        for (int i = 0; i < 2; i++) { //@2
            new Thread(() -> {
                System.out.println(Thread.currentThread() + "," + context.getBean("threadBean"));
                System.out.println(Thread.currentThread() + "," + context.getBean("threadBean"));
            }).start();
            TimeUnit.SECONDS.sleep(1);
        }
    }
}
```
