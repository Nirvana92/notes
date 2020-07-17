LinkedList: 

##### 基于JDK1.8版本

LinkedList 底层是通过双向链表来实现的. 

有个头结点[Node] first, 一个尾节点[Node] last.
通过属性 `int size` 来控制List的大小。

`Node first`, `Node last`, `int size` 都通过`transient`修饰. 表示序列化的时候不进行序列化.并且和ArrayList 一样, 通过readObject, writeObject 方法来自己实现序列化。

默认私有的 readObject() writeObject() 在序列化和反序列化的时候会被通过反射的方式调用。

ObjectOutputStream: 
```
public final void writeObject(Object obj) throws IOException {
        if (enableOverride) {
            writeObjectOverride(obj);
            return;
        }
        try {
            writeObject0(obj, false);
        } catch (IOException ex) {
            if (depth == 0) {
                writeFatalException(ex);
            }
            throw ex;
        }
    }
```
具体可以参考: `java.io.ObjectOutputStream#writeSerialData` 中的 `slotDesc.invokeWriteObject(obj, this);`


节点[Node]类: 
```
private static class Node<E> {
        // 节点的值
        E item;
        // 下一个节点
        Node<E> next;
        // 前一个节点
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

根据下标[index] 找节点, 根据index 和 size>>1[总大小的一半]来判断, 小于从头开始遍历. 大于等于从尾开始遍历。