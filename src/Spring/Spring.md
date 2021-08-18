## IOC

> https://javadoop.com/post/spring-ioc

1、到这里已经初始化了 Bean 容器，<bean /> 配置也相应的转换为了一个个 BeanDefinition，然后注册了各个 BeanDefinition 到注册中心，并且发送了注册事件。

说到这里，我们回到 refresh() 方法，我重新贴了一遍代码，看看我们说到哪了。是的，我们才说完 obtainFreshBeanFactory() 方法。

2、准备 Bean 容器: prepareBeanFactory。finishBeanFactoryInitialization(beanFactory); 这个巨头了，这里会负责初始化所有的 singleton beans。

FactoryBean 适用于 Bean 的创建过程比较复杂的场景，比如数据库连接池的创建。



----

ioc aop 的面试：
0、先自己看一下源码，留个印象
1、弄清楚流程，流向。关键步骤。关键词
2、常见面试题的八股文

第一步：配置元数据

Spring中配置元数据的方式有以下三种：xml java 注解

### XML启动

第二步：配置解析

完成元数据的配置之后，那么接下来就需要对配置进行解析，显而易见，不同配置方式的语法和对数据的描述是不一样的。
所以Spring封装了不同的配置方式的解析器，比如：
XML：使用ClasspathXmlApplicationContext作为入口，XmlBeanDefinitionReader进行Bean的元数据解析
Java代码、注解：使用AnnotationConfigApplicationContext作为入口，AnnotatedBeanDefinitionReader进行Bean的元数据解析

三：IOC中的几个关键类，分别是：
BeanFactory： 它是最顶层的一个接口，见名知意，典型的工厂模式。
ApplicationContext： 上面BeanFactory工厂帮我们拿到对象，那么工厂是如何产生对象的，那就要看它，它提供了具体的IOC实现，GenericApplicationContext、ClasspathXmlApplicationContext等，同时还提供了以下的附加服务：
BeanDefinition： Bean对象在Spring中的描述定义，可以理解为容器中每个Bean对象都是以BeanDefinition来保存的。
BeanDefinitionReader： 配置Bean的元数据的解析器，比如XmlBeanDefinitionReader、AnnotatedBeanDefinitionReader


我们主要看obtainFreshBeanFactory()方法，Bean的加载注册就是这里面实现的，之后的代码都是容器的初始化信息源和生命周期的一些事件。到这里Bean的资源文件就完成定位了，接下来我们就看加载的具体代码实现。

到这里，基于Xml的IOC容器初始化流程就已经全部完成了，基本看完整个流程之后也梳理了初始化的时序图以及流程图，来加深对整个容器初始化流程的理解，同时我们知道了IOC容器的初始化主要就三步：定位、加载、注册。

### 注解启动

Annotation来完成IOC容器的初始化。前面IOC源码解析中，我们已经知道第一步会调用scan()方法来扫描配置的指定包路径下的所有Bean对象，并封装成BeanDefinition对象存放beanDefinitionMap中，之后会调用refresh()方法去载入Bean的配置资源
```
public static void main(String[] args) {
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext("com.test.spring");
        AopA aopA = ac.getBean(AopA.class);
        aopA.print();
}
```

从顶层设计的层面来梳理一下整个IOC的初始化流程：
通过ResourceLoader从类路径、文件系统、URL等方式来定位资源文件位置；
配置Bean数据的文件会被抽象成Resource来处理；
容器通过BeanDefinitionReader来解析Resource，而实际处理过程是委托BeanDefinitionParserDelegate来完成的；
实现BeanDefinitionRegistry接口的子类将BeanDefinition注册到容器中；
容器的本质就是维护一个ConcurrentHashMap来保存BeanDefinition，后续Bean的操作都是围绕这个ConcurrentHashMap来实现的；
使用的时候我们通过BeanFactory和ApplicaitonContext来获得这些Bean对象。

## DI

DI（Dependency Injection）依赖注入：当对象内包含对其他对象的引用时，Spring会帮我们创建或定义这些依赖对象，而无需知道依赖对象的位置甚至具体实现类，实现更有效的解耦。

在IOC的初始化中其实包含了依赖注入的过程，同时通过对getBean()的延伸，进一步得到了第一个问题的答案，依赖注入的时机：
非延迟加载的Bean： IOC初始化中对进行预实例化（也是调用getBean()方法）。
延迟加载的Bean： 第一次调用getBean()方法时。

第二步：注入策略

Spring中注入策略有两种：

有参构造方法注入： 带有参数的构造函数来完成实例化或者带参数的static工厂方法，因为需要对参数进行解析转换，所以过程复杂。
无参构造方法注入： 默认实例化方法，例如使用setter方法注入，通过无参数构造函数或无参数的static工厂方法，过程简单。
第三步：自动装配方式

Spring官网中已经有详细的介绍，关于自动装配的方式有四种：

no：默认没有自动装配，必须由ref配置Bean的引用。
byName：根据属性名称在IOC容器中查找Bean，然后通过setter方法来注入属性（前提是必须有）。
byType：根据属性类型在IOC容器中查找Bean，只存在一个才会注入，多个会引发异常，没有则不注入。
constructor：根据构造函数的参数类型在IOC容器中查找Bean，没有则引发异常。

FactoryBean： 看到这个类，我们很容易想到BeanFactory，其实他两没有关系，IOC中我们已经了解BeanFactory是一个工厂的顶层接口，定义了IOC容器的基本行为，获取bean及bean的各种属性。而FactoryBean是一个工厂Bean接口，目的是用来隐藏复杂Bean的实例化细节，子类通过定制来实现实例化的逻辑，用来产生其他Bean实例。


### 依赖注入流程

所以，DI其实是 IOC 的一个部分。

0、我们知道它是一个顶层接口，实现肯定是在它的子类中。顺着准备工作中介绍的DefaultListableBeanFactory类中的preInstantiateSingletons()方法中调用的getBean()方法，点进去我们发现实际调用的是AbstractBeanFactory的getBean()方法。
1、转换真正的beanName
2.从缓存中获取Bean。1、“三级缓存”。2、第一步缓存没有拿到Bean的话，就会再从当前容器的存在的父容器的缓存
3.对依赖的Bean先进行初始化
4.根据不同Scope的Bean进行实例化(重点)
4.1.实例化Bean
4.2.提前缓存Bean
4.3.属性注入
4.4.初始化Bean
4.5.注册Bean
5.获取真正需要的Bean

验证了Bean的实例化入口离不开BeanFactory#getBean()方法，底层通过反射或者CGLIB代理完成对象创建，属性根据名称和类型自动注入等等。

## 循环依赖

从大到小的包含：1、ioc容器初始化。2、DI注入。3、循环依赖。

明白循环依赖及其解决的实现才是更深层次的了解Spring中Bean实例化过程的核心。同时Spring还有其他很多重要功能及实现，像AOP、BeanPostProcessor、Aware等等，如果说Bean是心脏，那它们更像是围绕着的血管一样前后遍布，加强扩展了Spring整体的功能。

有参构造方法注入

所以不管单例还是多例，这种有参构造方法注入（也就是Spring中说的构造函数注入）的场景导致的循环依赖在Spring中常规情况是无法解决的。但是Spring中允许通过一个非常规办法来绕开这个死结，就是@Lazy注解，待会我们再详细讲。

无参构造方法注入的场景，也是就Spring中说的setter注入，先准备测试类，
看来单例的构造器注入是正常的。我们再来看看原型类型，报错

@DependsOn注解
我们先介绍下@DependsOn注解的作用：任何指定此注解的依赖Bean都保证要在被依赖Bean之前由容器创建。

@Lazy的方案和逻辑

这时候返回的BeanA的代理对象就不是null了，所以BeanB也可以完成实例化，接着BeanA也完成实例化了。但是回到main方法内，在打印BeanB时，如果你打开debug，你会发现BeanB中的属性BeanA还是一个代理类，但是接着打印beanB.getBeanA()时发现它就是指向BeanA的引用地址。其实BeanB中的属性BeanA拿到真正BeanA实例化的引用就是在第一次调用getBeanA()的时候，它就会调用之前自定义封装的TargetSource的getTarget()的方法，最后还是通过调用getBean()方法，从缓存中拿到BeanA的实例。

## AOP

AOP（Aspect Oriented Programming）：面向切面编程，与面向对象编程OOP的关键单位是类不一样，它的关键单位是切面，它通过提供改变程序结构的方式来补充OOP。通俗点就是说我们可以通过预编译或者运行时动态代理在不修改方法源码的情况下增强方法的功能。

评论：刚刚才恍惚间发现，AOP的功能比 Node.js 里面之前说的 koa.js 的 middleware 的功能更强，middleware 其实更像是 hook 和函数，积木，加功能；而Java里的AOP，AspectJ可以直接字节码增强，动态性更强。

Spring中的代理机制分为两种：

JDK动态代理：内置在JDK中，通过拦截和反射来实现，被代理的对象必须要实现接口。
CGLIB动态代理：一个开源类的定义库，通过ASM字节码生成的方式生成代理类，默认代理没有实现的接口的对象，不能对final修饰的方法进行代理；Spring中可以通过设置@EnableAspectJAutoProxy(proxyTargetClass = true)强制使用用CGLIB进行动态代理。

我们知道 createAopProxy 方法有可能返回 JdkDynamicAopProxy 实例，也有可能返回 ObjenesisCglibAopProxy 实例，这里总结一下：

如果被代理的目标类实现了一个或多个自定义的接口，那么就会使用 JDK 动态代理，如果没有实现任何接口，会使用 CGLIB 实现代理，如果设置了 proxy-target-class="true"，那么都会使用 CGLIB。

JDK 动态代理基于接口，所以只有接口中的方法会被增强，而 CGLIB 基于类继承，需要注意就是如果方法使用了 final 修饰，或者是 private 方法，是不能被增强的。

而完成 AOP的初始化 其实也是穿插在IOC容器的初始化过程中。

BeanPostProcessor的作用是：定义了Bean的初始化回调方法，在其实例化、配置和初始化之后实现一些自定义逻辑。你可以理解为Bean对象的拦截器，如果你想扩展Bean的功能或对其修改包装等，就可以通过实现它去完成。

那我们为什么启动仅仅是通过配置类最终把AnnotationAwareAspectJAutoProxyCreator注册到容器中去呢？其实我们看下它的类图结构就知道了。

启动和调用流程：

1、最终完成自动代理的配置的生效启用。
2、通过配置的切面类来对匹配规则的目标Bean在其初始化之后对其做代理的相应处理。
解析@Aspect切面配置（获取容器中注册的所有beanName；
遍历所有beanName，并找到配置@Aspect注解的切面类；
解析获取切面类中配置的通知方法；
缓存最后获取的通知方法。）
3、生成代理对象，createProxy
4、调用代理方法。完成前后会调用invokeJoinpoint()方法，而invokeJoinpoint()方法的本质就是直接通过反射调用被代理类中的目标方法。




