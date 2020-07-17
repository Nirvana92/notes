ArrayList: 

#### 基于JDK1.8版本

ArrayList 默认初始大小 `DEFAULT_CAPACITY = 10`

ArrayList 默认初始化为空数组.
即: 

1. EMPTY_ELEMENTDATA[有参构造函数. initialCapacity=0]
2. DEFAULTCAPACITY_EMPTY_ELEMENTDATA[无参构造函数]


ArrayList 计算容量: 
1. minCapacity = size + 1;
2. 如果 elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA.
minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);

返回minCapacity;

ArrayList扩容问题: 
计算的minCapacity, 和elementData.length 判断,如果minCapacity > elementData.length. 进行扩容.

1. 首先之前容量进行1.5倍扩容.
2. 如果1.5扩容之后小于 期望容量[minCapacity], 则直接使用期望容量[minCapacity]
3. 如果扩容之后的容量大于最大数组容量[Integer.MAX_VALUE - 8]. 则进行期望容量[minCapacity] 是否大于最大数组容量[Integer.MAX_VALUE - 8]. 大于的话返回Integer.MAX_VALUE.小于返回最大数组容量.

然后通过Arrays.copy()对对象数组和新容量进行扩容操作。


问题: 
为什么ArrayList 最大的数组容量为Integer.MAX_VALUE-8.
因为某些虚拟机需要保留一些关键字。如果尝试分配超过此大小的内存可能会出现OOM.

为什么elementData 使用transient 修饰.
如果需要序列化, elementData 中的数据不全是需要序列化的[有空的数据存在]。
所以ArrayList 自己实现了序列化和反序列化的方法提供调用.[readObject, writeObject]