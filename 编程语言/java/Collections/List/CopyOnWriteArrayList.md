> CopyOnWriteArrayList 是一个线程安全的List.它是通过锁[cas]和copy来实现的。但是这种代价太高。
>
> 所以CopyOnWriteArrayList 比较适合多读少写的场景。能提高吞吐量。

`CopyOnWriteArrayList`的参数属性: 

```java
private transient volatile Object[] array;
// 获取array
final Object[] getArray() {
	return array;
}
// 设置array
final void setArray(Object[] a) {
	array = a;
}
```

再看看构造函数: 

```java
public CopyOnWriteArrayList() {
	setArray(new Object[0]);
}

public CopyOnWriteArrayList(Collection<? extends E> c) {
        Object[] elements;
        if (c.getClass() == CopyOnWriteArrayList.class)
            elements = ((CopyOnWriteArrayList<?>)c).getArray();
        else {
            elements = c.toArray();
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elements.getClass() != Object[].class)
                elements = Arrays.copyOf(elements, elements.length, Object[].class);
        }
        setArray(elements);
}
```

**问题: **下面数组生成的两种方式有没有什么本质区别

```java
// 方式1
Object[] objs = {};
// 方式2
Object[] objs = new Object[0];
```

**查看字节码: **上面两种方式的字节码都是下面这种

```
0 iconst_0
1 anewarray #2 <java/lang/Object>
4 astore_1
```

**问题: **第二个构造参数, 第一种方式, 直接将array赋值给本list.不会出现array使用冲突吗？

因为CopyOnWriteArrayList本来就是通过copy的方式进行array的操作。所以后续每次都会拷贝新的数组。不会产生冲突。

**问题: **第二个构造函数为什么会有`c.toArray might (incorrectly) not return Object[] (see 6260652)`提示.什么场景下返回的不是Object[].

```java
// asList这种生成List的方式, 在底层直接将具体类型的数组(如本例的String[])直接赋值给list中的Object[]
List<String> strs = Arrays.asList("a", "b");

Object[] objs = strs.toArray();
// 如果是上面初始化list的方式, 此时进行Object[] 的替换就会报错。s
// java.lang.ArrayStoreException: java.lang.Object
objs[0] = new Object();
```





