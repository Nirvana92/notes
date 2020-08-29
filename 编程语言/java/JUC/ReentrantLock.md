`ReentrantLock`是基于`AQS`实现的锁机制。

首先看下 `ReentrantLock`中实现的同步器: 

```java
abstract static class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = -5179523762034025860L;

        /**
         * Performs {@link Lock#lock}. The main reason for subclassing
         * is to allow fast path for nonfair version.
         */
  			// 锁方法
        abstract void lock();

        /**
         * Performs non-fair tryLock.  tryAcquire is implemented in
         * subclasses, but both need nonfair try for trylock method.
         */
  			// 这个非公平锁的实现为什么没放入到 Sync 的非公平实现 NonfairSync 中。
  			// 如果当前没有线程占用锁: 尝试获取锁
  			// 如果有线程占用锁: 查看占用锁的线程是否是当前线程, 如果是进行可重入操作 state++
  			// 其他情况返回尝试失败; 返回false
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
				// 尝试释放
        protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }
  			// .... 后面方法省略
    }
```

非公平锁的实现: 

```java
static final class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;

        /**
         * Performs lock.  Try immediate barge, backing up to normal
         * acquire on failure.
         */
  			// 锁方法: 首先尝试通过cas 获取锁; 获取成功将锁的归属线程设置为ownerThread
  			// 尝试获取锁失败; 调用acquire()[aqs方法] 方法
        final void lock() {
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
                acquire(1);
        }
				// aqs 的acquire() 方法最终会调用 tryAcquire() 方法.
        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
    }
```

公平锁的实现: 

```java
static final class FairSync extends Sync {
        private static final long serialVersionUID = -3000897897090466540L;

        final void lock() {
            acquire(1);
        }

        /**
         * Fair version of tryAcquire.  Don't grant access unless
         * recursive call or no waiters or is first.
         */
  			// 如果当前没有线程占用锁: 判断是否有前置节点: 没有前置节点尝试获取锁: 获取锁成功将锁占有的线程设置为锁的持有者
  			// 判断锁的线程持有者是否是当前线程; 如果是: state ++
        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }
```

