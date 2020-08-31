`volatile特性`

1. 可见性： 在JMM模型中, 可以保证volatile 修饰的在多线程之间修改在多线程可见。
2. 有序性: 禁止指令重排序.

`volatile` 写-读的内存语义: 

`volatile写: `当写线程写一个`volatile`变量, JMM会把该线程对应的本地内存中的共享变量值刷新到主内存。

`volatile读: `当读线程写一个`volatile`变量时, JMM会把该线程对应的本地内存置为无效, 线程接下来将从主内存读取共享变量。

**`volatile`可见性实现原理:** 

```java
public class BeanInstance {
  private static BeanInstance instance = null;
  private BeanInstance() {}
  
	public static BeanInstance getInstance() {
    if(instance == null) {
      instance = new BeanInstance();
    }
    
    return instance;
  }
  
  public static void main(String[] args) {
    BeanInstance.getInstance();
  }
}
```



查看`volatile`修饰的汇编指令可以发现, 会增加一个 `lock`修饰, 使得写`volatile`具有以下两个行为: 

1. 写`volatile`时处理器会将缓存写回到主内存。
2. 一个处理器的缓存写回到内存会导致其他处理器的缓存失效。

`volatile`有序性的实现原理[加内存屏障]: 

1. 在每个volatile写操作的前面插入一个StoreStore屏障，防止写volatile与后面的写操作重排序。
2. 在每个volatile写操作的后面插入一个StoreLoad屏障，防止写volatile与后面的读操作重排序。
3. 在每个volatile读操作的后面插入一个LoadLoad屏障，防止读volatile与后面的读操作重排序。
4. 在每个volatile读操作的后面插入一个LoadStore屏障，防止读volatile与后面的写操作重排序。

