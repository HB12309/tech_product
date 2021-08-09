- 使用 Random 类时，为了避免重复创建的开销，我们一般将实例化好的 Random 对象设置为我们所使用服务对象的属性或静态属性
- 这在线程竞争不激烈的情况下没有问题，但在一个高并发的 web 服务内，使用同一个 Random 对象可能会导致线程阻塞。

1.同一个种子,生成N个随机数,当你设定种子的时候,这N个随机数是什么已经确定。相同次数生成的随机数字是完全相同的。
2.如果用相同的种子创建两个 Random 实例,如上面的r3,r4,则对每个实例进行相同的方法调用序列,它们将生成并返回相同的数字序列。
3.Java的随机数都是通过算法实现的,Math.random()本质上属于Random()类。
4.使用java.util.Random()会相对来说比较灵活一些。

RandomUtils 的底层，还是使用了 util.Random

```
import org.apache.commons.lang3.RandomUtils;
```

## ThreadLocalRandom
```
import java.util.concurrent.ThreadLocalRandom;

public class ThreadLocalRandom extends Random {}
```

后面我开始研究一个 Class 或者框架的时候，自己提问题好了。

1、 Random 是怎么做到随机的？是否是线程安全？高并发下有什么问题？如何解决？所谓的`伪随机`体现在哪里？

1.伪随机数是看似随机实质是固定的周期性序列,也就是有规则的随机。
2.只要这个随机数是由确定算法生成的,那就是伪随机,只能通过不断算法优化,使你的随机数更接近随机。随机这个属性和算法本身就是矛盾的
3.通过真实随机事件取得的随机数才是真随机数。
4.Random 的随机原理是对一个”随机种子”进行固定的算术和位运算，得到随机结果，再使用这个结果作为下一次随机的种子。在解决线程安全问题时，Random 使用 CAS 更新下一次随机的种子，可以想到，如果多个线程同时使用这个对象，就肯定会有一些线程执行 CAS 连续失败，进而导致线程阻塞。(其实代码就是看 next(int bits))

基本算法：linear congruential pseudorandom number generator (LGC) 线性同余法伪随机数生成器
缺点：可预测
在注重信息安全的应用中，不要使用 LCG 算法生成随机数，请使用 SecureRandom。



2、ThreadLocalRandom 在什么package下，解决了 Random 什么问题？

它是出现了JUC 里面，不是在 util 也不是在 lang 下

- ThreadLocalRandom 的实现需要 Thread 对象的配合，在 Thread 对象内存在着一个属性 threadLocalRandomSeed，它保存着这个线程专属的随机种子，而这个属性在 Thread 对象的 offset，是在 ThreadLocalRandom 类加载时就确定了的
- 具体方法是 SEED = UNSAFE.objectFieldOffset(Thread.class.getDeclaredField("threadLocalRandomSeed"));
- 一个对象所占用的内存大小在类被加载后就确定了的，所以使用 Unsafe.objectFieldOffset(class, fieldName) 可以获取到某个属性在类中偏移量，而在找对了偏移量，又能确定数据类型时，使用 ThreadLocalRandom 就是很安全的。
- 每一个线程有一个独立的随机数生成器，用于并发产生随机数，能够解决多个线程发生的竞争争夺。效率更高！

3、SecurityRandom 是为了解决什么问题出现的 ？

```
package java.Security.SecureRandom

public class SecureRandom extends java.util.Random {}

```

操作系统收集了一些随机事件，比如鼠标点击，键盘点击等等，SecureRandom 使用这些随机事件作为种子。

SecureRandom 提供加密的强随机数生成器 (RNG)，要求种子必须是不可预知的，产生非确定性输出。
SecureRandom 也提供了与实现无关的算法，因此，调用方（应用程序代码）会请求特定的 RNG 算法并将它传回到该算法的 SecureRandom 对象中。


### 随机字符串

可以使用 Apache Commons-Lang 包中的 RandomStringUtils 类。
Maven 依赖如下：
```
<dependency>
    <groupId>commons-lang</groupId>
    <artifactId>commons-lang</artifactId>
    <version>2.6</version>
</dependency>
```