线程中的中断: 

`Thread` 中关于中断的方法: 

`public boolean isInterrupted(): `实例方法, 用于判断当前实例线程是否处于中断状态。

`public void interrupt()`: 实例方法, 对实例线程修改中断标识. 标记调用线程状态为中断。

```java
public boolean isInterrupted() {
  return isInterrupted(false);
}
```

`public static boolean interrupted(): `静态方法, 对当前线程进行中断。即修改当前线程的状态标识为中断。

```java
public static boolean interrupted() {
  return currentThread().isInterrupted(true);
}
```

上面两个方法最终调用本地方法: 

```java
/**
	* Tests if some Thread has been interrupted.  The interrupted state
	* is reset or not based on the value of ClearInterrupted that is
	* passed.
	*/
// ClearInterrupted: 表示是否清楚中断状态
private native boolean isInterrupted(boolean ClearInterrupted);
```

代码案例: 

```java
class InterThread implements Runnable {
    @SneakyThrows
    @Override
    public void run() {
      	// 输出true, true
				System.out.println(Thread.currentThread().isInterrupted());
        System.out.println(Thread.currentThread().isInterrupted());
       	// 输出true, false; 调用静态方法对中断状态进行了重置
				// System.out.println(Thread.interrupted());
				// System.out.println(Thread.interrupted());
    }
}
```

调用代码: 

```java
public static void main(String[] args) {
        Thread thread = new Thread(new InterThread());
        thread.start();
        thread.interrupt();	
    }
```

`Thread`的一些方法: 

```java
// 告知调度者当前线程可以让出执行; 调度者可以随意忽略此请求
public static native void yield();
// 线程睡眠
public static native void sleep(long millis);
// 该线程可以执行; JVM虚拟机会调用线程的run方法; 结果是两个线程同时执行; 一个线程执行start方法, 另一个执行run方法
public synchronized void start();
// 如果当前线程通过runnable 构造; 则会执行runnable 中的run方法; 如果不是通过runnable构造则什么也不做直接返回
public void run();
// 该方法交给系统执行; 作用是在真正结束之前完成清理工作
private void exit();
// 强制终止线程
// 非安全的; 所以过期处理: 
// 调用stop会释放所有已经获取的monitors ;
public final void stop();
// 中断线程
public void interrupt();
// 挂起线程
public final void suspend();
// 恢复一个挂起线程
public final void resume();
// 
public static native boolean holdsLock(Object obj);
```

