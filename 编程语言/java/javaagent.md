`AQS: AbstractQueuedSynchronizer`

> 提供了一个框架, 通过先进先出的等待队列。通过原子性的state 控制状态。

如果需要通过aqs作为同步器的基础。可以使用 `getState`和`setState`检查状态来重新定义如下方法[下面方法均抛出`UnsupportedOperationException`]: 

`tryAcquire`

`tryRelease`

`tryAcquireShared`

`tryReleaseShared`

`isHeldExclusively`

`AQS`中的`Node` 是`CLH`的一种变体: 

`CLH锁: ` CLH是一种基于单向链表(隐式创建)的高性能、公平的自旋锁，申请加锁的线程只需要在其前驱节点的本地变量上自旋，从而极大地减少了不必要的处理器缓存同步的次数，降低了总线和内存的开销。

`MCS锁: ` MCS 的实现是基于链表的，每个申请锁的线程都是链表上的一个节点，这些线程会一直轮询自己的本地变量，来知道它自己是否获得了锁。已经获得了锁的线程在释放锁的时候，负责通知其它线程，这样 CPU 之间缓存的同步操作就减少了很多，仅在线程通知另外一个线程的时候发生，降低了系统总线和内存的开销。

`CLH锁`和`MCS锁`比较: 

CLH 锁与 MCS 锁的原理大致相同，都是各个线程轮询各自关注的变量，来避免多个线程对同一个变量的轮询，从而从 CPU 缓存一致性的角度上减少了系统的消耗。
CLH 锁与 MCS 锁最大的不同是，MCS 轮询的是当前队列节点的变量，而 CLH 轮询的是当前节点的前驱节点的变量，来判断前一个线程是否释放了锁。

`AQS`中的Node 等待状态: 

```
// 取消的状态标识
// 当前node 因为超时或者中断被取消, 当处于当前状态不会再转变到其他状态。而且当前状态的线程不会再次被阻塞
static final int CANCELLED =  1;
// 标明后继线程等待唤醒
// 当前节点的后继节点阻塞或即将阻塞。当前节点 releases或者cancels 需要unpark 当前节点的后继节点。
static final int SIGNAL    = -1;
// 标明线程处于等待条件
static final int CONDITION = -2;
// 标明下一个acquireShared 应该无条件传播
static final int PROPAGATE = -3;
```



`ConditionObject` 实现了`Condition`

> AQS中的ConditionObject 维护了一个链表

```java
public class ConditionObject implements Condition, java.io.Serializable {
        /** First node of condition queue. */
        private transient Node firstWaiter;
        /** Last node of condition queue. */
        private transient Node lastWaiter;
}
```

