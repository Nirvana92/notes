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

