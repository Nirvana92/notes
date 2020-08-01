通过跳表实现的Map结构. 线程安全.

Node结构: 

```java
Node(K key, Object value, Node<K,V> next) {
   // key值
   this.key = key;
   // 值
   this.value = value;
   // 下一个指针指向的node
   this.next = next;
}
```

调表头结点结构Index: 

```java
static class Index<K,V> {
  final Node<K,V> node;
  final Index<K,V> down;
  volatile Index<K,V> right;
}
```

跳表每层头结点结构`HeadIndex`:

```java
static final class HeadIndex<K,V> extends Index<K,V> {
    final int level;
    HeadIndex(Node<K,V> node, Index<K,V> down, Index<K,V> right, int level) {
        super(node, down, right);
        this.level = level;
    }
}
```

 