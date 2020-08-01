#### 基于JDK1.8版本

> 允许key和value为null

构造方法: 
`HashMap(int initialCapacity, float loadFactor)`

`initialCapacity`: HashMap初始容量.
判断initialCapacity不小于0 并且 如果大于 1<<30 则直接赋值1<<30.

通过`tableSizeFor(initialCapacity)` 计算扩容阈值. threshold
threshold: 下次扩容size达到的阈值.
计算公式: `(capacity * load factor)`

`tableSizeFor(initialCapacity)`: 返回>= initialCapacity 的最小的二的倍数的值。比如initialCapacity=7, 返回8. initialCapacity=9, 返回16;

负载因子需要大于0 并且通过NaN[Not-a-Number]校验.

问题: HashMap中的阈值[threshold], 默认初始容量[DEFAULT_INITIAL_CAPACITY] 为什么都必须是2的倍数。
tab[i = (n - 1) & hash]  n-1 二进制上都是1, 更好的减少hash碰撞。

`HashMap` 的hash()函数. 
1. 如果key == null. 返回0;
2: 如果非空, key.hashCode() 高16位和低16位异或。


`HashMap`的`put()`操作: 
1. table==null || table.length==0, 进行resize()操作
2. table通过key定位Node是否为空. 如果为空新增一个Node.
3. 如果定位的Node不为空.
       3.1. 如果当前的key == tab[index].key 或者当前key.equals(tab[index].key)
       3.2. 如果当前的当前的Node 是TreeNode
       3.3. 否则[不满足3.1, 3.2] 遍历tab[index]到尾, 然后新增一个node添加进去。
              3.3.1. 期间如果tab[index]上的节点数>=TREEIFY_THRESHOLD[8],执行treeifyBin()
              3.3.2. 如果table.length.也就是table数组的长度<MIN_TREEIFY_CAPACITY[64]. 进行resize()扩容操作。
              3.3.3. 否则将table[index]这条链上的节点转成红黑树。

   ​	3.4. 如果在上述过程中找到任何一个key值相等或者equals的。直接进行value 的值替换。


问题: 为什么在putVal()的时候要把table 抽出一个临时变量tab.不能直接在table上操作。

`resize()`主要做的操作: 
初始化新的newCap[容量, 即table.length], newThr[阈值]

1. 如果oldCap > 0; 
    1.1. oldCap >= MAXIMUM_CAPACITY. 阈值=Integer.max_value;并返回.不再进行扩容。
    1.2. 否则, newCap = oldCap <<1; newThr=oldThr<<1;
2. 否则，如果oldThr>0: newCap=oldThr.初始容量等于阈值。
3. 否则: newCap=16. newThr=0.75*16;
然后遍历oldTab, 进行新的newTab重新hash重新赋值。

问题: 这些集合为什么要添加`modCount`和fast-fail机制.

安全保护机制。