#### 基于JDK1.8 版本

也是一个key-value 的容器。

> 不允许key或value 为null

`Hashtable`要求key必须实现hashCode()和equals()方法.

HashTable 定位下标index, 是通过(key.hashCode & 0x7FFFFFFF) % tab.length. 的方式。
也证明了需要保证key 不为null。

HashTable 和HashMap 的最大的不同是HashTable方法都添加了`synchronized`修饰。

问题: `HashTable`通过Map初始化的时候为什么取map.size*2 和 11的最大值。