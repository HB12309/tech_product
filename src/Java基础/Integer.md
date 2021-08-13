# int 的包装类

我们仔细读下，其实就会发现他其实是计算传入的int型x的位数，要求x为正数。

```java
final static int [] sizeTable = { 9, 99, 999, 9999, 99999, 999999, 9999999,99999999, 999999999, Integer.MAX_VALUE };

// Requires positive x
static int stringSize1(int x) {
    for (int i=0; ; i++)
        if (x <= sizeTable[i])
            return i+1;
}

static int myStringSize1(int x){
    return String.valueOf(x).length();
}

static int myStringSize2(int x){
   int num=1;
   while(x>10){
        x=x/10;
        num++;
   }
    return num;
}

// Requires positive x
static int stringSize2(int x) {
    int p = 10;
    for (int i=1; i<11; i++) {
        if (x < p)
            return i;
        p = 10*p;
    }
    return 10;
}
```

评：一个需求，可以有多种实现方式，从时间和空间的角度，再进行压力测试，找出最快的的方法。在 JDK 里面有代码都有很高的质量，可以学习。比如一个字段的复用，比如bit位计算等。

## Integer的内部类 IntegerCache

很容易理解这段代码，初始化Integer后，IntegerCache会缓存[-128,127]之间的数据，这个区间的上限可以配置，取决于java.lang.Integer.IntegerCache.high这个属性，这个属性在VM参数里为-XX:AutoBoxCacheMax=2000进行设置调整或者VM里设置-Djava.lang.Integer.IntegerCache.high=2000。所以Integer在初始化完成后会缓存[-128,max]之间的数据。

这个是逻辑，使用的时候在下面，就是缓存呗，减少 new Integer

```java
public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```
