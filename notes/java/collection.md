<!-- TOC -->
- [一、接口继承关系和实现](#一接口继承关系和实现)
- [二、List](#二List)
  - [ArrayList](#ArrayList)
  - [Vector](#Vector)
  - [Linkedlist](#Linkedlist)
- [三、Set](#三Set)
  - [HashSet](#HashSet)
  - [TreeSet](#TreeSet)
  - [LinkedHashSet](#LinkedHashSet)
- [四、Map](#四Map)
  - [HashMap](#HashMap)
  - [ConcurrentHashMap](#ConcurrentHashMap)
  - [HashTable](#HashTable)
  - [TreeMap](#TreeMap)
  - [LinkedHashMap](#LinkedHashMap)
<!-- TOC -->


# 一、接口继承关系和实现

# 二、List

## List

List 是有序的集合（元素添加顺序）。

ArrayList

ArrayList 内部是通过数组实现，可以对元素进行快速随机访问；初始容量为10，增长因子为1.5。
在 JDK 1.7 的时候，默认的初始化容量在无参构造函数中给出，在 JDK 1.8 的时候，默认容量是在第一次调用 add 方法的时候设置的。

注意：下文中没有特殊注明的话，都是 JDK 1.8 的源码。

```Java
private static final int DEFAULT_CAPACITY = 10;

transient Object[] elementData;

public E get(int index) {
 rangeCheck(index);
 return elementData(index);
}
```

看到这里不仅有个疑问了，ArrayList 实现了 Serializable 接口，而这里的 elementData 由被 transient 修饰，那 ArrayList 到底能不能被序列化呢？测试一下，是可以序列化的（测试代码就不贴了）。

再让我们回到 ArrayList 的源码，可以看到 ArrayList 是通过 writeObject() 和 readObject() 这两个方法来实现序列化的；
这样做的是因为 ArrayList 中的数据 elementData 中的有效元素个数不一定等于数组的 size （动态扩容的机制），所以通过对 elementData 中的有效元素进行序列化，可以提高效率。

这里说到了动态扩容机制，那我们也看看相关的源码：

```Java
/**
 * Increases the capacity to ensure that it can hold at least the
 * number of elements specified by the minimum capacity argument.
 *
 * @param minCapacity the desired minimum capacity
 */
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

从这段代码中可以看到，当数组容量不够的时候会进行扩容，扩容后的数组长度为原长度的1.5倍，并且扩容的过程是新建一个（原长度 * 1.5）的数据，再把旧数组中的有效元素拷贝过去：

```Java
int newCapacity = oldCapacity + (oldCapacity >> 1);

相当于：
int newCapacity = oldCapacity + (oldCapacity / 2); //原长度的1.5倍
```

```Java
Arrays.copyOf(elementData, newCapacity);

底层实现为：
public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
        T[] copy = ((Object)newType == (Object)Object[].class)
            ? (T[]) new Object[newLength]
            : (T[]) Array.newInstance(newType.getComponentType(), newLength);
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
}
```

因为 ArrayList 的底层是基于数据实现的，所以从 ArrayList 中间位置插入或者删除元素时，需要对数组进行复制、移动，效率会比较低，所以 ArrayList 适合随机查找和遍历，不适合插入和删除。

## Vector

Vector 底层也是通过数据实现的，不过它所有的 public 方法都增加了 synchronized ，以确保线程同步，避免多线程同时操作引起不一致性，因此它的访问速度会比 ArrayList 低。

```Java
public synchronized boolean add(E e)
```

另外，Vector 的扩容机制和 ArrayList 不同：

1. ArrayList 有两个属性：elementData 存数据的数组，size 数组中元素的个数。
2. Vector 有三个属性：elementData 存数据的数组，elementCount 存数组中元素的个数，capacityIncrement 为扩容因子。

ArrayList 扩容的时候直接扩大原长度的 1.5 倍，而 Vector 的扩容要和扩容因子有关：当扩容因子为 0 时 ，Vector 的容量翻倍，而当扩容因子大于 0 时，新数组长度为原数组长度+扩容因子大小。

```Java
int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                         capacityIncrement : oldCapacity);
```

这里还有一个问题，为什么 Vector 的序列化，不像 ArrayList 那样实现？查阅了一些资料，大部分说法都是认为 Vector 是早期的容器，基本上已经废弃不用了，依然保留是为了兼容性，所以没有必要再优化维护了。


## Linkedlist

LinkedList 底层是用链表结构来进行数据存储；链表的插入和删除操作很快，而随机查询和遍历的速度比较慢；

```Java
private static class Node<E> {
 E item;
 Node<E> next;
 Node<E> prev;

 Node(Node<E> prev, E element, Node<E> next) {
     this.item = element;
     this.next = next;
     this.prev = prev;
 }
}
```

因为是链表的数据结构，那么也不涉及到扩容的问题，每次新增元素的时候，直接在链表中添加一个节点；LinkedList 还有专门的方法可以操作表头和表尾元素，所以 LinkedList 可以当做栈和队列使用。

# 三、Set

## HashSet

## TreeSet

## LinkedHashSet

# 四、Map

## HashMap

## ConcurrentHashMap

## HashTable

## TreeMap

## LinkedHashMap
