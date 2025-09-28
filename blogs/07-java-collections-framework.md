# Java集合框架的设计哲学与性能优化

## 引言

Java集合框架是Java程序设计中最重要的工具包之一，它提供了一套性能优良、使用方便的数据结构和算法。从简单的List到复杂的ConcurrentHashMap，Java集合框架体现了Java语言的设计智慧和工程实践。在这篇文章中，我将深入剖析Java集合框架的设计哲学、核心实现、性能优化以及最佳实践。

## 1. 集合框架概述

### 1.1 集合框架的体系结构

```java
// Java集合框架的核心接口层次
public class CollectionsFrameworkHierarchy {
    // Collection接口层次
    // Collection (根接口)
    // ├── List (有序集合，可重复)
    // │   ├── ArrayList
    // │   ├── LinkedList
    // │   └── Vector (已过时)
    // ├── Set (无序集合，不可重复)
    // │   ├── HashSet
    // │   ├── LinkedHashSet
    // │   └── TreeSet
    // └── Queue (队列)
    //     ├── LinkedList
    //     ├── PriorityQueue
    //     └── ArrayDeque

    // Map接口层次 (独立于Collection)
    // Map (键值对映射)
    // ├── HashMap
    // ├── LinkedHashMap
    // ├── TreeMap
    // ├── Hashtable (已过时)
    // └── ConcurrentHashMap

    // 集合框架的设计哲学：
    // 1. 接口与实现分离
    // 2. 统一的操作方式
    // 3. 灵活的扩展机制
    // 4. 性能与功能的平衡
}
```

### 1.2 集合框架的设计原则

```java
public class DesignPrinciples {
    // 1. 接口与实现分离
    public void interfaceImplementationSeparation() {
        // 面向接口编程
        List<String> list = new ArrayList<>();  // 可以随时切换实现
        Set<String> set = new HashSet<>();
        Map<String, Integer> map = new HashMap<>();

        // 这种设计允许在不修改客户端代码的情况下更换实现
        // 只需要改变实例化部分即可
    }

    // 2. 统一的操作方式
    public void unifiedOperations() {
        // 所有集合都支持相似的操作
        Collection<String> collection1 = new ArrayList<>();
        Collection<String> collection2 = new HashSet<>();

        // 统一的添加操作
        collection1.add("item1");
        collection2.add("item2");

        // 统一的遍历方式
        for (String item : collection1) {
            System.out.println(item);
        }

        for (String item : collection2) {
            System.out.println(item);
        }

        // 统一的迭代器
        Iterator<String> iterator1 = collection1.iterator();
        Iterator<String> iterator2 = collection2.iterator();
    }

    // 3. 算法与数据结构分离
    public void algorithmDataStructureSeparation() {
        List<Integer> numbers = Arrays.asList(3, 1, 4, 1, 5, 9, 2, 6);

        // Collections工具类提供算法
        Collections.sort(numbers);  // 排序算法
        Collections.reverse(numbers);  // 反转算法
        Collections.shuffle(numbers);  // 随机打乱

        // 算法可以应用于不同的集合实现
        Set<Integer> set = new HashSet<>(numbers);
        List<Integer> sortedList = new ArrayList<>(set);
        Collections.sort(sortedList);
    }

    // 4. 泛型与类型安全
    public void genericsAndTypeSafety() {
        // 编译时类型检查
        List<String> stringList = new ArrayList<>();
        stringList.add("Hello");
        // stringList.add(123);  // 编译错误

        // 泛型方法
        List<Integer> intList = Arrays.asList(1, 2, 3);
        List<Double> doubleList = Arrays.asList(1.0, 2.0, 3.0);

        // 统一的处理方式
        processList(stringList);
        processList(intList);
    }

    public <T> void processList(List<T> list) {
        for (T item : list) {
            System.out.println(item);
        }
    }
}
```

## 2. List接口深度解析

### 2.1 ArrayList实现原理

```java
public class ArrayListDeepDive {
    // ArrayList的内部结构
    private static class MyArrayList<E> {
        private static final int DEFAULT_CAPACITY = 10;
        private Object[] elementData;
        private int size;

        public MyArrayList() {
            this.elementData = new Object[DEFAULT_CAPACITY];
            this.size = 0;
        }

        public MyArrayList(int initialCapacity) {
            if (initialCapacity > 0) {
                this.elementData = new Object[initialCapacity];
            } else if (initialCapacity == 0) {
                this.elementData = new Object[DEFAULT_CAPACITY];
            } else {
                throw new IllegalArgumentException("Illegal Capacity: " + initialCapacity);
            }
            this.size = 0;
        }

        // 添加元素的核心逻辑
        public boolean add(E e) {
            ensureCapacityInternal(size + 1);  // 确保容量足够
            elementData[size++] = e;
            return true;
        }

        private void ensureCapacityInternal(int minCapacity) {
            if (elementData.length < minCapacity) {
                grow(minCapacity);
            }
        }

        // 扩容机制
        private void grow(int minCapacity) {
            int oldCapacity = elementData.length;
            int newCapacity = oldCapacity + (oldCapacity >> 1);  // 1.5倍扩容
            if (newCapacity < minCapacity) {
                newCapacity = minCapacity;
            }
            elementData = Arrays.copyOf(elementData, newCapacity);
        }

        // 在指定位置添加元素
        public void add(int index, E element) {
            rangeCheckForAdd(index);
            ensureCapacityInternal(size + 1);
            System.arraycopy(elementData, index, elementData, index + 1, size - index);
            elementData[index] = element;
            size++;
        }

        private void rangeCheckForAdd(int index) {
            if (index > size || index < 0) {
                throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
            }
        }

        private String outOfBoundsMsg(int index) {
            return "Index: " + index + ", Size: " + size;
        }

        // 获取元素
        @SuppressWarnings("unchecked")
        public E get(int index) {
            rangeCheck(index);
            return (E) elementData[index];
        }

        private void rangeCheck(int index) {
            if (index >= size) {
                throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
            }
        }

        // 删除元素
        public E remove(int index) {
            rangeCheck(index);
            E oldValue = (E) elementData[index];
            int numMoved = size - index - 1;
            if (numMoved > 0) {
                System.arraycopy(elementData, index + 1, elementData, index, numMoved);
            }
            elementData[--size] = null;  // 清除引用，帮助GC
            return oldValue;
        }

        public int size() {
            return size;
        }
    }

    // ArrayList的性能特性：
    // 1. 随机访问快：O(1)
    // 2. 插入删除慢：O(n)
    // 3. 内存连续性好
    // 4. 扩容成本较高

    // ArrayList的使用场景：
    // 1. 频繁随机访问
    // 2. 很少在中间插入删除
    // 3. 遍历操作较多
    // 4. 大小相对稳定
}
```

### 2.2 LinkedList实现原理

```java
public class LinkedListDeepDive {
    // LinkedList的内部结构
    private static class MyLinkedList<E> {
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

        private Node<E> first;
        private Node<E> last;
        private int size;

        public MyLinkedList() {
        }

        // 在头部添加元素
        public void addFirst(E e) {
            final Node<E> f = first;
            final Node<E> newNode = new Node<>(null, e, f);
            first = newNode;
            if (f == null) {
                last = newNode;
            } else {
                f.prev = newNode;
            }
            size++;
        }

        // 在尾部添加元素
        public void addLast(E e) {
            final Node<E> l = last;
            final Node<E> newNode = new Node<>(l, e, null);
            last = newNode;
            if (l == null) {
                first = newNode;
            } else {
                l.next = newNode;
            }
            size++;
        }

        // 添加元素
        public boolean add(E e) {
            addLast(e);
            return true;
        }

        // 在指定位置添加元素
        public void add(int index, E element) {
            checkPositionIndex(index);

            if (index == size) {
                addLast(element);
            } else {
                addBefore(element, node(index));
            }
        }

        private void addBefore(E e, Node<E> succ) {
            final Node<E> pred = succ.prev;
            final Node<E> newNode = new Node<>(pred, e, succ);
            succ.prev = newNode;
            if (pred == null) {
                first = newNode;
            } else {
                pred.next = newNode;
            }
            size++;
        }

        // 获取指定位置的节点
        Node<E> node(int index) {
            if (index < (size >> 1)) {
                Node<E> x = first;
                for (int i = 0; i < index; i++) {
                    x = x.next;
                }
                return x;
            } else {
                Node<E> x = last;
                for (int i = size - 1; i > index; i--) {
                    x = x.prev;
                }
                return x;
            }
        }

        // 删除元素
        public E remove(int index) {
            checkElementIndex(index);
            return unlink(node(index));
        }

        E unlink(Node<E> x) {
            final E element = x.item;
            final Node<E> next = x.next;
            final Node<E> prev = x.prev;

            if (prev == null) {
                first = next;
            } else {
                prev.next = next;
                x.prev = null;
            }

            if (next == null) {
                last = prev;
            } else {
                next.prev = prev;
                x.next = null;
            }

            x.item = null;
            size--;
            return element;
        }

        // 获取元素
        public E get(int index) {
            checkElementIndex(index);
            return node(index).item;
        }

        private void checkElementIndex(int index) {
            if (!isElementIndex(index)) {
                throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
            }
        }

        private void checkPositionIndex(int index) {
            if (!isPositionIndex(index)) {
                throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
            }
        }

        private boolean isElementIndex(int index) {
            return index >= 0 && index < size;
        }

        private boolean isPositionIndex(int index) {
            return index >= 0 && index <= size;
        }

        private String outOfBoundsMsg(int index) {
            return "Index: " + index + ", Size: " + size;
        }

        public int size() {
            return size;
        }

        // 实现Deque接口的方法
        public void push(E e) {
            addFirst(e);
        }

        public E pop() {
            return removeFirst();
        }

        public E removeFirst() {
            final Node<E> f = first;
            if (f == null) {
                throw new NoSuchElementException();
            }
            return unlinkFirst(f);
        }

        private E unlinkFirst(Node<E> f) {
            final E element = f.item;
            final Node<E> next = f.next;
            f.item = null;
            f.next = null;
            first = next;
            if (next == null) {
                last = null;
            } else {
                next.prev = null;
            }
            size--;
            return element;
        }
    }

    // LinkedList的性能特性：
    // 1. 随机访问慢：O(n)
    // 2. 插入删除快：O(1)
    // 3. 内存开销大
    // 4. 可以作为队列和栈使用

    // LinkedList的使用场景：
    // 1. 频繁的插入删除操作
    // 2. 需要队列或栈功能
    // 3. 很少随机访问
    // 4. 大小经常变化
}
```

### 2.3 ArrayList vs LinkedList

```java
public class ListComparison {
    public void performanceComparison() {
        final int size = 100000;
        final int warmup = 10000;

        // 准备数据
        List<Integer> arrayList = new ArrayList<>();
        List<Integer> linkedList = new LinkedList<>();

        for (int i = 0; i < size; i++) {
            arrayList.add(i);
            linkedList.add(i);
        }

        // 预热
        for (int i = 0; i < warmup; i++) {
            arrayList.get(i % size);
            linkedList.get(i % size);
        }

        // 测试随机访问性能
        long startTime = System.nanoTime();
        for (int i = 0; i < size; i++) {
            arrayList.get(i);
        }
        long arrayListRandomAccess = System.nanoTime() - startTime;

        startTime = System.nanoTime();
        for (int i = 0; i < size; i++) {
            linkedList.get(i);
        }
        long linkedListRandomAccess = System.nanoTime() - startTime;

        // 测试中间插入性能
        startTime = System.nanoTime();
        for (int i = 0; i < 1000; i++) {
            arrayList.add(size / 2, i);
        }
        long arrayListInsert = System.nanoTime() - startTime;

        startTime = System.nanoTime();
        for (int i = 0; i < 1000; i++) {
            linkedList.add(size / 2, i);
        }
        long linkedListInsert = System.nanoTime() - startTime;

        // 输出性能比较结果
        System.out.println("随机访问性能:");
        System.out.println("ArrayList: " + arrayListRandomAccess / 1000000.0 + " ms");
        System.out.println("LinkedList: " + linkedListRandomAccess / 1000000.0 + " ms");
        System.out.println("ArrayList比LinkedList快 " + (double) linkedListRandomAccess / arrayListRandomAccess + " 倍");

        System.out.println("\n中间插入性能:");
        System.out.println("ArrayList: " + arrayListInsert / 1000000.0 + " ms");
        System.out.println("LinkedList: " + linkedListInsert / 1000000.0 + " ms");
        System.out.println("LinkedList比ArrayList快 " + (double) arrayListInsert / linkedListInsert + " 倍");
    }

    // 选择List实现的决策树
    public void chooseListImplementation() {
        // 需要随机访问吗？
        //   是 -> 使用ArrayList
        //   否 -> 需要频繁在头部/尾部操作吗？
        //          是 -> 使用LinkedList
        //          否 -> 考虑ArrayList

        // 需要线程安全吗？
        //   是 -> 使用Collections.synchronizedList()
        //   否 -> 根据上述选择

        // 内存敏感吗？
        //   是 -> 使用ArrayList
        //   否 -> 根据功能需求选择
    }
}
```

## 3. Set接口深度解析

### 3.1 HashSet实现原理

```java
import java.util.*;

public class HashSetDeepDive {
    // HashSet的内部结构
    private static class MyHashSet<E> {
        private static final int DEFAULT_INITIAL_CAPACITY = 16;
        private static final float DEFAULT_LOAD_FACTOR = 0.75f;

        private transient HashMap<E, Object> map;
        private static final Object PRESENT = new Object();

        public MyHashSet() {
            map = new HashMap<>(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
        }

        public MyHashSet(int initialCapacity) {
            map = new HashMap<>(initialCapacity);
        }

        public boolean add(E e) {
            return map.put(e, PRESENT) == null;
        }

        public boolean remove(Object o) {
            return map.remove(o) == PRESENT;
        }

        public boolean contains(Object o) {
            return map.containsKey(o);
        }

        public int size() {
            return map.size();
        }

        public Iterator<E> iterator() {
            return map.keySet().iterator();
        }
    }

    // HashSet的内部实现
    private static class InternalHashSet<E> {
        private static final int DEFAULT_CAPACITY = 16;
        private static final float LOAD_FACTOR = 0.75f;

        private Node<E>[] table;
        private int size;
        private int threshold;
        private final float loadFactor;

        private static class Node<E> {
            final int hash;
            final E key;
            Node<E> next;

            Node(int hash, E key, Node<E> next) {
                this.hash = hash;
                this.key = key;
                this.next = next;
            }
        }

        public InternalHashSet() {
            this(DEFAULT_CAPACITY, LOAD_FACTOR);
        }

        public InternalHashSet(int initialCapacity, float loadFactor) {
            if (initialCapacity <= 0) {
                throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
            }
            if (loadFactor <= 0 || Float.isNaN(loadFactor)) {
                throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
            }
            this.loadFactor = loadFactor;
            this.table = new Node[initialCapacity];
            this.threshold = (int) (initialCapacity * loadFactor);
        }

        // 添加元素的核心逻辑
        public boolean add(E e) {
            if (e == null) {
                throw new NullPointerException();
            }

            int hash = hash(e);
            int index = (table.length - 1) & hash;

            Node<E> node = table[index];
            if (node == null) {
                table[index] = new Node<>(hash, e, null);
                size++;
                if (size >= threshold) {
                    resize();
                }
                return true;
            } else {
                Node<E> current = node;
                while (current != null) {
                    if (current.hash == hash && (current.key == e || current.key.equals(e))) {
                        return false;  // 元素已存在
                    }
                    if (current.next == null) {
                        current.next = new Node<>(hash, e, null);
                        size++;
                        if (size >= threshold) {
                            resize();
                        }
                        return true;
                    }
                    current = current.next;
                }
            }
            return false;
        }

        // 计算哈希值
        private int hash(Object key) {
            int h = key.hashCode();
            // 确保哈希值分布均匀
            return h ^ (h >>> 16);
        }

        // 扩容机制
        private void resize() {
            Node<E>[] oldTable = table;
            int oldCapacity = oldTable.length;
            int newCapacity = oldCapacity << 1;  // 扩容为2倍
            Node<E>[] newTable = new Node[newCapacity];

            threshold = (int) (newCapacity * loadFactor);

            for (int i = 0; i < oldCapacity; i++) {
                Node<E> node = oldTable[i];
                if (node != null) {
                    Node<E> next;
                    do {
                        next = node.next;
                        int index = (newCapacity - 1) & node.hash;
                        node.next = newTable[index];
                        newTable[index] = node;
                        node = next;
                    } while (node != null);
                }
            }

            table = newTable;
        }

        // 查找元素
        public boolean contains(Object o) {
            if (o == null) {
                return false;
            }
            int hash = hash(o);
            int index = (table.length - 1) & hash;

            Node<E> node = table[index];
            while (node != null) {
                if (node.hash == hash && (node.key == o || node.key.equals(o))) {
                    return true;
                }
                node = node.next;
            }
            return false;
        }

        public int size() {
            return size;
        }
    }

    // HashSet的性能特性：
    // 1. 添加、删除、查找的平均时间复杂度：O(1)
    // 2. 最坏情况（哈希冲突严重）：O(n)
    // 3. 无序存储
    // 4. 允许null元素

    // HashSet的使用场景：
    // 1. 需要快速查找
    // 2. 不需要保持顺序
    // 3. 元素不重复
    // 4. 大小可变
}
```

### 3.2 LinkedHashSet实现原理

```java
public class LinkedHashSetDeepDive {
    // LinkedHashSet继承自HashSet，通过LinkedHashMap实现
    private static class MyLinkedHashSet<E> {
        private static class Node<E> {
            E key;
            Node<E> before;
            Node<E> after;

            Node(E key, Node<E> before, Node<E> after) {
                this.key = key;
                this.before = before;
                this.after = after;
            }
        }

        private static final int DEFAULT_INITIAL_CAPACITY = 16;
        private static final float DEFAULT_LOAD_FACTOR = 0.75f;

        private Node<E>[] table;
        private Node<E> header;  // 双向链表头节点
        private int size;
        private int threshold;
        private final float loadFactor;

        public MyLinkedHashSet() {
            this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
        }

        public MyLinkedHashSet(int initialCapacity, float loadFactor) {
            if (initialCapacity <= 0) {
                throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
            }
            if (loadFactor <= 0 || Float.isNaN(loadFactor)) {
                throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
            }
            this.loadFactor = loadFactor;
            this.table = new Node[initialCapacity];
            this.threshold = (int) (initialCapacity * loadFactor);

            // 初始化双向链表
            header = new Node<>(null, null, null);
            header.before = header;
            header.after = header;
        }

        public boolean add(E e) {
            if (e == null) {
                throw new NullPointerException();
            }

            int hash = hash(e);
            int index = (table.length - 1) & hash;

            Node<E> node = table[index];
            if (node == null) {
                // 创建新节点并插入链表
                Node<E> newNode = new Node<>(hash, e, null);
                table[index] = newNode;
                addToLinkedList(newNode);
                size++;
                if (size >= threshold) {
                    resize();
                }
                return true;
            } else {
                Node<E> current = node;
                while (current != null) {
                    if (current.hash == hash && (current.key == e || current.key.equals(e))) {
                        return false;  // 元素已存在
                    }
                    if (current.next == null) {
                        // 在链表末尾添加新节点
                        Node<E> newNode = new Node<>(hash, e, null);
                        current.next = newNode;
                        addToLinkedList(newNode);
                        size++;
                        if (size >= threshold) {
                            resize();
                        }
                        return true;
                    }
                    current = current.next;
                }
            }
            return false;
        }

        // 添加到双向链表
        private void addToLinkedList(Node<E> node) {
            node.after = header;
            node.before = header.before;
            header.before.after = node;
            header.before = node;
        }

        // 从双向链表移除
        private void removeFromLinkedList(Node<E> node) {
            node.before.after = node.after;
            node.after.before = node.before;
        }

        private int hash(Object key) {
            int h = key.hashCode();
            return h ^ (h >>> 16);
        }

        private void resize() {
            Node<E>[] oldTable = table;
            int oldCapacity = oldTable.length;
            int newCapacity = oldCapacity << 1;
            Node<E>[] newTable = new Node[newCapacity];

            threshold = (int) (newCapacity * loadFactor);

            for (int i = 0; i < oldCapacity; i++) {
                Node<E> node = oldTable[i];
                while (node != null) {
                    Node<E> next = node.next;
                    int index = (newCapacity - 1) & node.hash;
                    node.next = newTable[index];
                    newTable[index] = node;
                    node = next;
                }
            }

            table = newTable;
        }

        public Iterator<E> iterator() {
            return new LinkedHashSetIterator();
        }

        private class LinkedHashSetIterator implements Iterator<E> {
            private Node<E> next = header.after;
            private Node<E> lastReturned;

            public boolean hasNext() {
                return next != header;
            }

            public E next() {
                if (next == header) {
                    throw new NoSuchElementException();
                }
                lastReturned = next;
                next = next.after;
                return lastReturned.key;
            }

            public void remove() {
                if (lastReturned == null) {
                    throw new IllegalStateException();
                }
                MyLinkedHashSet.this.remove(lastReturned.key);
                lastReturned = null;
            }
        }

        public boolean remove(Object o) {
            if (o == null) {
                return false;
            }
            int hash = hash(o);
            int index = (table.length - 1) & hash;

            Node<E> prev = null;
            Node<E> current = table[index];
            while (current != null) {
                if (current.hash == hash && (current.key == o || current.key.equals(o))) {
                    if (prev == null) {
                        table[index] = current.next;
                    } else {
                        prev.next = current.next;
                    }
                    removeFromLinkedList(current);
                    size--;
                    return true;
                }
                prev = current;
                current = current.next;
            }
            return false;
        }

        public int size() {
            return size;
        }
    }

    // LinkedHashSet的性能特性：
    // 1. 插入、删除、查找的平均时间复杂度：O(1)
    // 2. 保持插入顺序
    // 3. 比HashSet略慢（需要维护链表）
    // 4. 内存开销略大

    // LinkedHashSet的使用场景：
    // 1. 需要保持插入顺序
    // 2. 需要快速查找
    // 3. 元素不重复
    // 4. 需要按插入顺序遍历
}
```

### 3.3 TreeSet实现原理

```java
public class TreeSetDeepDive {
    // TreeSet基于TreeMap实现，使用红黑树
    private static class MyTreeSet<E> {
        private static final class TreeNode<E> {
            E key;
            TreeNode<E> left;
            TreeNode<E> right;
            TreeNode<E> parent;
            boolean red;  // 红黑树的颜色属性

            TreeNode(E key, TreeNode<E> parent) {
                this.key = key;
                this.parent = parent;
                this.red = true;  // 新节点默认为红色
            }
        }

        private TreeNode<E> root;
        private final Comparator<? super E> comparator;
        private int size;

        public MyTreeSet() {
            this.comparator = null;
        }

        public MyTreeSet(Comparator<? super E> comparator) {
            this.comparator = comparator;
        }

        // 添加元素
        public boolean add(E e) {
            if (e == null) {
                throw new NullPointerException();
            }

            TreeNode<E> t = root;
            if (t == null) {
                root = new TreeNode<>(e, null);
                root.red = false;  // 根节点为黑色
                size = 1;
                return true;
            }

            int cmp;
            TreeNode<E> parent;
            Comparator<? super E> cpr = comparator;

            if (cpr != null) {
                do {
                    parent = t;
                    cmp = cpr.compare(e, t.key);
                    if (cmp < 0) {
                        t = t.left;
                    } else if (cmp > 0) {
                        t = t.right;
                    } else {
                        return false;  // 元素已存在
                    }
                } while (t != null);
            } else {
                // 使用自然排序
                Comparable<? super E> comparable = (Comparable<? super E>) e;
                do {
                    parent = t;
                    cmp = comparable.compareTo(t.key);
                    if (cmp < 0) {
                        t = t.left;
                    } else if (cmp > 0) {
                        t = t.right;
                    } else {
                        return false;  // 元素已存在
                    }
                } while (t != null);
            }

            // 创建新节点
            TreeNode<E> newNode = new TreeNode<>(e, parent);
            if (cmp < 0) {
                parent.left = newNode;
            } else {
                parent.right = newNode;
            }

            // 修复红黑树性质
            fixAfterInsertion(newNode);
            size++;
            return true;
        }

        // 修复红黑树性质（插入后）
        private void fixAfterInsertion(TreeNode<E> x) {
            x.red = true;  // 新节点为红色

            while (x != null && x != root && x.parent.red) {
                if (parentOf(x) == leftOf(parentOf(parentOf(x)))) {
                    TreeNode<E> y = rightOf(parentOf(parentOf(x)));
                    if (colorOf(y) == true) {
                        setColor(parentOf(x), false);
                        setColor(y, false);
                        setColor(parentOf(parentOf(x)), true);
                        x = parentOf(parentOf(x));
                    } else {
                        if (x == rightOf(parentOf(x))) {
                            x = parentOf(x);
                            rotateLeft(x);
                        }
                        setColor(parentOf(x), false);
                        setColor(parentOf(parentOf(x)), true);
                        rotateRight(parentOf(parentOf(x)));
                    }
                } else {
                    TreeNode<E> y = leftOf(parentOf(parentOf(x)));
                    if (colorOf(y) == true) {
                        setColor(parentOf(x), false);
                        setColor(y, false);
                        setColor(parentOf(parentOf(x)), true);
                        x = parentOf(parentOf(x));
                    } else {
                        if (x == leftOf(parentOf(x))) {
                            x = parentOf(x);
                            rotateRight(x);
                        }
                        setColor(parentOf(x), false);
                        setColor(parentOf(parentOf(x)), true);
                        rotateLeft(parentOf(parentOf(x)));
                    }
                }
            }

            root.red = false;  // 确保根节点为黑色
        }

        // 辅助方法
        private static <E> boolean colorOf(TreeNode<E> p) {
            return (p == null ? false : p.red);
        }

        private static <E> TreeNode<E> parentOf(TreeNode<E> p) {
            return (p == null ? null : p.parent);
        }

        private static <E> TreeNode<E> leftOf(TreeNode<E> p) {
            return (p == null ? null : p.left);
        }

        private static <E> TreeNode<E> rightOf(TreeNode<E> p) {
            return (p == null ? null : p.right);
        }

        private static <E> void setColor(TreeNode<E> p, boolean c) {
            if (p != null) {
                p.red = c;
            }
        }

        // 左旋
        private void rotateLeft(TreeNode<E> p) {
            if (p != null) {
                TreeNode<E> r = p.right;
                p.right = r.left;
                if (r.left != null) {
                    r.left.parent = p;
                }
                r.parent = p.parent;
                if (p.parent == null) {
                    root = r;
                } else if (p.parent.left == p) {
                    p.parent.left = r;
                } else {
                    p.parent.right = r;
                }
                r.left = p;
                p.parent = r;
            }
        }

        // 右旋
        private void rotateRight(TreeNode<E> p) {
            if (p != null) {
                TreeNode<E> l = p.left;
                p.left = l.right;
                if (l.right != null) {
                    l.right.parent = p;
                }
                l.parent = p.parent;
                if (p.parent == null) {
                    root = l;
                } else if (p.parent.right == p) {
                    p.parent.right = l;
                } else {
                    p.parent.left = l;
                }
                l.right = p;
                p.parent = l;
            }
        }

        // 查找元素
        public boolean contains(Object o) {
            return getNode(o) != null;
        }

        private TreeNode<E> getNode(Object o) {
            if (o == null) {
                throw new NullPointerException();
            }

            TreeNode<E> p = root;
            Comparator<? super E> cpr = comparator;

            if (cpr != null) {
                while (p != null) {
                    int cmp = cpr.compare((E) o, p.key);
                    if (cmp < 0) {
                        p = p.left;
                    } else if (cmp > 0) {
                        p = p.right;
                    } else {
                        return p;
                    }
                }
            } else {
                Comparable<? super E> comparable = (Comparable<? super E>) o;
                while (p != null) {
                    int cmp = comparable.compareTo(p.key);
                    if (cmp < 0) {
                        p = p.left;
                    } else if (cmp > 0) {
                        p = p.right;
                    } else {
                        return p;
                    }
                }
            }

            return null;
        }

        // 中序遍历
        public Iterator<E> iterator() {
            return new TreeSetIterator();
        }

        private class TreeSetIterator implements Iterator<E> {
            private TreeNode<E> next = firstNode();
            private TreeNode<E> lastReturned;

            private TreeNode<E> firstNode() {
                TreeNode<E> p = root;
                if (p != null) {
                    while (p.left != null) {
                        p = p.left;
                    }
                }
                return p;
            }

            public boolean hasNext() {
                return next != null;
            }

            public E next() {
                if (next == null) {
                    throw new NoSuchElementException();
                }
                lastReturned = next;
                next = successor(next);
                return lastReturned.key;
            }

            private TreeNode<E> successor(TreeNode<E> t) {
                if (t == null) {
                    return null;
                }

                if (t.right != null) {
                    TreeNode<E> p = t.right;
                    while (p.left != null) {
                        p = p.left;
                    }
                    return p;
                } else {
                    TreeNode<E> p = t.parent;
                    TreeNode<E> ch = t;
                    while (p != null && ch == p.right) {
                        ch = p;
                        p = p.parent;
                    }
                    return p;
                }
            }
        }

        public int size() {
            return size;
        }
    }

    // TreeSet的性能特性：
    // 1. 插入、删除、查找的时间复杂度：O(log n)
    // 2. 保持自然排序或自定义排序
    // 3. 有序存储
    // 4. 不允许null元素

    // TreeSet的使用场景：
    // 1. 需要有序存储
    // 2. 需要范围查询
    // 3. 元素不重复
    // 4. 需要排序功能
}
```

## 4. Map接口深度解析

### 4.1 HashMap实现原理

```java
import java.util.*;

public class HashMapDeepDive {
    // HashMap的内部结构
    private static class MyHashMap<K, V> {
        private static final int DEFAULT_INITIAL_CAPACITY = 16;
        private static final float DEFAULT_LOAD_FACTOR = 0.75f;
        private static final int MAXIMUM_CAPACITY = 1 << 30;
        private static final int TREEIFY_THRESHOLD = 8;
        private static final int UNTREEIFY_THRESHOLD = 6;

        private Node<K, V>[] table;
        private int size;
        private int threshold;
        private final float loadFactor;

        // 基本节点结构
        static class Node<K, V> implements Map.Entry<K, V> {
            final int hash;
            final K key;
            V value;
            Node<K, V> next;

            Node(int hash, K key, V value, Node<K, V> next) {
                this.hash = hash;
                this.key = key;
                this.value = value;
                this.next = next;
            }

            public K getKey() { return key; }
            public V getValue() { return value; }
            public V setValue(V newValue) {
                V oldValue = value;
                value = newValue;
                return oldValue;
            }
        }

        // 红黑树节点结构
        static final class TreeNode<K, V> extends Node<K, V> {
            TreeNode<K, V> parent;
            TreeNode<K, V> left;
            TreeNode<K, V> right;
            TreeNode<K, V> prev;
            boolean red;

            TreeNode(int hash, K key, V val, Node<K, V> next) {
                super(hash, key, val, next);
            }
        }

        public MyHashMap() {
            this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
        }

        public MyHashMap(int initialCapacity, float loadFactor) {
            if (initialCapacity < 0) {
                throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
            }
            if (initialCapacity > MAXIMUM_CAPACITY) {
                initialCapacity = MAXIMUM_CAPACITY;
            }
            if (loadFactor <= 0 || Float.isNaN(loadFactor)) {
                throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
            }
            this.loadFactor = loadFactor;
            this.threshold = tableSizeFor(initialCapacity);
        }

        // 计算合适的容量
        private static int tableSizeFor(int cap) {
            int n = -1 >>> Integer.numberOfLeadingZeros(cap - 1);
            return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
        }

        // 计算哈希值
        static final int hash(Object key) {
            int h;
            return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
        }

        // 放入元素的核心逻辑
        public V put(K key, V value) {
            return putVal(hash(key), key, value, false, true);
        }

        private V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
            Node<K, V>[] tab;
            Node<K, V> p;
            int n, i;

            // 如果表为空或长度为0，进行初始化
            if ((tab = table) == null || (n = tab.length) == 0) {
                n = (tab = resize()).length;
            }

            // 计算索引位置，如果该位置为空，直接插入
            if ((p = tab[i = (n - 1) & hash]) == null) {
                tab[i] = newNode(hash, key, value, null);
            } else {
                Node<K, V> e;
                K k;

                // 如果该位置有节点，检查是否相同key
                if (p.hash == hash &&
                    ((k = p.key) == key || (key != null && key.equals(k)))) {
                    e = p;
                } else if (p instanceof TreeNode) {
                    // 如果是树节点，使用树的put方法
                    e = ((TreeNode<K, V>) p).putTreeVal(this, tab, hash, key, value);
                } else {
                    // 遍历链表
                    for (int binCount = 0; ; ++binCount) {
                        if ((e = p.next) == null) {
                            p.next = newNode(hash, key, value, null);
                            if (binCount >= TREEIFY_THRESHOLD - 1) {
                                treeifyBin(tab, hash);
                            }
                            break;
                        }
                        if (e.hash == hash &&
                            ((k = e.key) == key || (key != null && key.equals(k)))) {
                            break;
                        }
                        p = e;
                    }
                }

                // 如果找到了相同的key，更新value
                if (e != null) {
                    V oldValue = e.value;
                    if (!onlyIfAbsent || oldValue == null) {
                        e.value = value;
                    }
                    return oldValue;
                }
            }

            ++modCount;
            if (++size > threshold) {
                resize();
            }
            return null;
        }

        // 创建新节点
        Node<K, V> newNode(int hash, K key, V value, Node<K, V> next) {
            return new Node<>(hash, key, value, next);
        }

        // 链表转树
        final void treeifyBin(Node<K, V>[] tab, int hash) {
            int n, index;
            Node<K, V> e;

            if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY) {
                resize();
            } else if ((e = tab[index = (n - 1) & hash]) != null) {
                TreeNode<K, V> hd = null, tl = null;
                do {
                    TreeNode<K, V> p = replacementTreeNode(e, null);
                    if (tl == null) {
                        hd = p;
                    } else {
                        p.prev = tl;
                        tl.next = p;
                    }
                    tl = p;
                } while ((e = e.next) != null);
                if ((tab[index] = hd) != null) {
                    hd.treeify(tab);
                }
            }
        }

        TreeNode<K, V> replacementTreeNode(Node<K, V> p, Node<K, V> next) {
            return new TreeNode<>(p.hash, p.key, p.value, next);
        }

        private static final int MIN_TREEIFY_CAPACITY = 64;

        // 扩容机制
        final Node<K, V>[] resize() {
            Node<K, V>[] oldTab = table;
            int oldCap = (oldTab == null) ? 0 : oldTab.length;
            int oldThr = threshold;
            int newCap, newThr = 0;

            // 计算新的容量和阈值
            if (oldCap > 0) {
                if (oldCap >= MAXIMUM_CAPACITY) {
                    threshold = Integer.MAX_VALUE;
                    return oldTab;
                } else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                         oldCap >= DEFAULT_INITIAL_CAPACITY) {
                    newThr = oldThr << 1;
                }
            } else if (oldThr > 0) {
                newCap = oldThr;
            } else {
                newCap = DEFAULT_INITIAL_CAPACITY;
                newThr = (int) (DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
            }

            if (newThr == 0) {
                float ft = (float) newCap * loadFactor;
                newThr = (newCap < MAXIMUM_CAPACITY && ft < MAXIMUM_CAPACITY ?
                          (int) ft : Integer.MAX_VALUE);
            }
            threshold = newThr;

            // 创建新表并重新散列
            Node<K, V>[] newTab = (Node<K, V>[]) new Node[newCap];
            table = newTab;

            if (oldTab != null) {
                for (int j = 0; j < oldCap; ++j) {
                    Node<K, V> e;
                    if ((e = oldTab[j]) != null) {
                        oldTab[j] = null;
                        if (e.next == null) {
                            newTab[e.hash & (newCap - 1)] = e;
                        } else if (e instanceof TreeNode) {
                            ((TreeNode<K, V>) e).split(this, newTab, j, oldCap);
                        } else {
                            Node<K, V> loHead = null, loTail = null;
                            Node<K, V> hiHead = null, hiTail = null;
                            Node<K, V> next;
                            do {
                                next = e.next;
                                if ((e.hash & oldCap) == 0) {
                                    if (loTail == null) {
                                        loHead = e;
                                    } else {
                                        loTail.next = e;
                                    }
                                    loTail = e;
                                } else {
                                    if (hiTail == null) {
                                        hiHead = e;
                                    } else {
                                        hiTail.next = e;
                                    }
                                    hiTail = e;
                                }
                            } while ((e = next) != null);
                            if (loTail != null) {
                                loTail.next = null;
                                newTab[j] = loHead;
                            }
                            if (hiTail != null) {
                                hiTail.next = null;
                                newTab[j + oldCap] = hiHead;
                            }
                        }
                    }
                }
            }
            return newTab;
        }

        // 获取元素
        public V get(Object key) {
            Node<K, V> e;
            return (e = getNode(hash(key), key)) == null ? null : e.value;
        }

        private Node<K, V> getNode(int hash, Object key) {
            Node<K, V>[] tab;
            Node<K, V> first, e;
            int n;
            K k;

            if ((tab = table) != null && (n = tab.length) > 0 &&
                (first = tab[(n - 1) & hash]) != null) {
                if (first.hash == hash &&
                    ((k = first.key) == key || (key != null && key.equals(k)))) {
                    return first;
                }
                if ((e = first.next) != null) {
                    if (first instanceof TreeNode) {
                        return ((TreeNode<K, V>) first).getTreeNode(hash, key);
                    }
                    do {
                        if (e.hash == hash &&
                            ((k = e.key) == key || (key != null && key.equals(k)))) {
                            return e;
                        }
                    } while ((e = e.next) != null);
                }
            }
            return null;
        }

        public int size() {
            return size;
        }

        transient int modCount = 0;
    }

    // HashMap的性能特性：
    // 1. 插入、删除、查找的平均时间复杂度：O(1)
    // 2. 最坏情况（哈希冲突严重）：O(n)
    // 3. Java 8+引入红黑树优化
    // 4. 允许null键和null值

    // HashMap的使用场景：
    // 1. 需要快速查找
    // 2. 键值对映射
    // 3. 不需要保持顺序
    // 4. 大小可变
}
```

### 4.2 ConcurrentHashMap实现原理

```java
import java.util.concurrent.*;
import java.util.concurrent.locks.*;
import java.util.*;

public class ConcurrentHashMapDeepDive {
    // ConcurrentHashMap的简化实现
    private static class MyConcurrentHashMap<K, V> {
        private static final int DEFAULT_CAPACITY = 16;
        private static final float DEFAULT_LOAD_FACTOR = 0.75f;
        private static final int CONCURRENCY_LEVEL = 16;
        private static final int MAX_SEGMENTS = 1 << 16;

        // 分段锁实现（Java 7及之前）
        private final Segment<K, V>[] segments;
        private final int segmentShift;
        private final int segmentMask;

        @SuppressWarnings("unchecked")
        public MyConcurrentHashMap(int initialCapacity, float loadFactor, int concurrencyLevel) {
            if (concurrencyLevel > MAX_SEGMENTS) {
                concurrencyLevel = MAX_SEGMENTS;
            }

            int sshift = 0;
            int ssize = 1;
            while (ssize < concurrencyLevel) {
                ++sshift;
                ssize <<= 1;
            }

            this.segmentShift = 32 - sshift;
            this.segmentMask = ssize - 1;
            this.segments = Segment.newArray(ssize);

            if (initialCapacity > MAXIMUM_CAPACITY) {
                initialCapacity = MAXIMUM_CAPACITY;
            }

            int c = initialCapacity / ssize;
            if (c * ssize < initialCapacity) {
                ++c;
            }

            int cap = MIN_SEGMENT_CAPACITY;
            while (cap < c) {
                cap <<= 1;
            }

            for (int i = 0; i < this.segments.length; ++i) {
                this.segments[i] = new Segment<K, V>(cap, loadFactor);
            }
        }

        // 分段类
        static final class Segment<K, V> extends ReentrantLock {
            private static final int MAX_SCAN_RETRIES = 2;
            private static final int MIN_SEGMENT_CAPACITY = 2;

            transient volatile HashEntry<K, V>[] table;
            transient int threshold;
            transient final float loadFactor;

            Segment(int initialCapacity, float lf) {
                this.loadFactor = lf;
                this.table = newTable(initialCapacity);
                this.threshold = (int) (initialCapacity * loadFactor);
            }

            @SuppressWarnings("unchecked")
            static final <K, V> HashEntry<K, V>[] newTable(int size) {
                return (HashEntry<K, V>[]) new HashEntry[size];
            }

            final V put(K key, int hash, V value, boolean onlyIfAbsent) {
                lock();
                try {
                    int c = count;
                    if (c++ > threshold) {
                        rehash();
                    }

                    HashEntry<K, V>[] tab = table;
                    int index = (tab.length - 1) & hash;
                    HashEntry<K, V> first = tab[index];

                    for (HashEntry<K, V> e = first; e != null; e = e.next) {
                        if (e.hash == hash && key.equals(e.key)) {
                            V oldValue = e.value;
                            if (!onlyIfAbsent) {
                                e.value = value;
                            }
                            return oldValue;
                        }
                    }

                    HashEntry<K, V> newEntry = new HashEntry<K, V>(key, hash, first, value);
                    tab[index] = newEntry;
                    count = c;
                    return null;
                } finally {
                    unlock();
                }
            }

            private void rehash() {
                HashEntry<K, V>[] oldTable = table;
                int oldCapacity = oldTable.length;
                if (oldCapacity >= MAX_SEGMENT_CAPACITY) {
                    return;
                }

                HashEntry<K, V>[] newTable = newTable(oldCapacity << 1);
                threshold = (int) (newTable.length * loadFactor);
                int sizeMask = newTable.length - 1;

                for (int i = 0; i < oldCapacity; i++) {
                    HashEntry<K, V> e = oldTable[i];
                    if (e != null) {
                        HashEntry<K, V> next = e.next;
                        int idx = e.hash & sizeMask;

                        if (next == null) {
                            newTable[idx] = e;
                        } else {
                            HashEntry<K, V> lastRun = e;
                            int lastIdx = idx;
                            for (HashEntry<K, V> last = next; last != null; last = last.next) {
                                int k = last.hash & sizeMask;
                                if (k != lastIdx) {
                                    lastIdx = k;
                                    lastRun = last;
                                }
                            }
                            newTable[lastIdx] = lastRun;

                            for (HashEntry<K, V> p = e; p != lastRun; p = p.next) {
                                int k = p.hash & sizeMask;
                                HashEntry<K, V> n = newTable[k];
                                newTable[k] = new HashEntry<K, V>(p.key, p.hash, n, p.value);
                            }
                        }
                    }
                }

                table = newTable;
            }

            transient volatile int count;
        }

        // 哈希条目
        static final class HashEntry<K, V> {
            final K key;
            final int hash;
            volatile V value;
            volatile HashEntry<K, V> next;

            HashEntry(K key, int hash, HashEntry<K, V> next, V value) {
                this.key = key;
                this.hash = hash;
                this.next = next;
                this.value = value;
            }
        }

        // 计算段索引
        private Segment<K, V> segmentFor(int hash) {
            return segments[(hash >>> segmentShift) & segmentMask];
        }

        // 放入元素
        public V put(K key, V value) {
            if (key == null || value == null) {
                throw new NullPointerException();
            }

            int hash = hash(key.hashCode());
            Segment<K, V> segment = segmentFor(hash);
            return segment.put(key, hash, value, false);
        }

        private int hash(int h) {
            h += (h << 15) ^ 0xffffcd7d;
            h ^= (h >>> 10);
            h += (h << 3);
            h ^= (h >>> 6);
            h += (h << 2) + (h << 14);
            return h ^ (h >>> 16);
        }

        // 获取元素
        public V get(Object key) {
            if (key == null) {
                throw new NullPointerException();
            }

            int hash = hash(key.hashCode());
            Segment<K, V> segment = segmentFor(hash);
            return segment.get(key, hash);
        }

        static final int MAXIMUM_CAPACITY = 1 << 30;

        @SuppressWarnings("unchecked")
        static final <K, V> Segment<K, V>[] newArray(int size) {
            return (Segment<K, V>[]) new Segment[size];
        }
    }

    // Java 8+的ConcurrentHashMap改进：
    private static class Java8ConcurrentHashMap<K, V> {
        private static final int DEFAULT_CAPACITY = 16;
        private static final float LOAD_FACTOR = 0.75f;
        private static final int TREEIFY_THRESHOLD = 8;
        private static final int UNTREEIFY_THRESHOLD = 6;

        private transient volatile Node<K, V>[] table;
        private transient volatile int sizeCtl;

        // 基本节点
        static class Node<K, V> {
            final int hash;
            final K key;
            volatile V val;
            volatile Node<K, V> next;

            Node(int hash, K key, V val, Node<K, V> next) {
                this.hash = hash;
                this.key = key;
                this.val = val;
                this.next = next;
            }
        }

        // CAS操作
        private static final sun.misc.Unsafe UNSAFE;
        private static final long SIZECTL;

        static {
            try {
                UNSAFE = sun.misc.Unsafe.getUnsafe();
                SIZECTL = UNSAFE.objectFieldOffset(Java8ConcurrentHashMap.class.getDeclaredField("sizeCtl"));
            } catch (Exception e) {
                throw new Error(e);
            }
        }

        // 放入元素
        public V put(K key, V value) {
            return putVal(key, value, false);
        }

        final V putVal(K key, V value, boolean onlyIfAbsent) {
            if (key == null || value == null) {
                throw new NullPointerException();
            }

            int hash = spread(key.hashCode());
            int binCount = 0;
            Node<K, V>[] tab = table;

            while (true) {
                Node<K, V> f;
                int n, i, fh;

                if (tab == null || (n = tab.length) == 0) {
                    tab = initTable();
                } else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                    if (casTabAt(tab, i, null, new Node<K, V>(hash, key, value, null))) {
                        break;
                    }
                } else if ((fh = f.hash) == MOVED) {
                    tab = helpTransfer(tab, f);
                } else {
                    V oldVal = null;
                    synchronized (f) {
                        if (tabAt(tab, i) == f) {
                            if (fh >= 0) {
                                binCount = 1;
                                for (Node<K, V> e = f; ; ++binCount) {
                                    K ek;
                                    if (e.hash == hash &&
                                        ((ek = e.key) == key ||
                                         (ek != null && key.equals(ek)))) {
                                        oldVal = e.val;
                                        if (!onlyIfAbsent) {
                                            e.val = value;
                                        }
                                        break;
                                    }
                                    Node<K, V> pred = e;
                                    if ((e = e.next) == null) {
                                        pred.next = new Node<K, V>(hash, key, value, null);
                                        break;
                                    }
                                }
                            }
                        }
                    }
                    if (binCount != 0) {
                        if (binCount >= TREEIFY_THRESHOLD) {
                            treeifyBin(tab, i);
                        }
                        if (oldVal != null) {
                            return oldVal;
                        }
                        break;
                    }
                }
            }

            addCount(1L, binCount);
            return null;
        }

        private final Node<K, V>[] initTable() {
            Node<K, V>[] tab;
            int sc;

            while ((tab = table) == null || tab.length == 0) {
                if ((sc = sizeCtl) < 0) {
                    Thread.yield();
                } else if (UNSAFE.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                    try {
                        if ((tab = table) == null || tab.length == 0) {
                            int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                            @SuppressWarnings("unchecked")
                            Node<K, V>[] nt = (Node<K, V>[]) new Node<?, ?>[n];
                            table = tab = nt;
                            sc = n - (n >>> 2);
                        }
                    } finally {
                        sizeCtl = sc;
                    }
                    break;
                }
            }
            return tab;
        }

        private static int spread(int h) {
            return (h ^ (h >>> 16)) & HASH_BITS;
        }

        private static final int HASH_BITS = 0x7fffffff;
        private static final int MOVED = -1;

        private static final sun.misc.Unsafe getUnsafe() {
            try {
                return sun.misc.Unsafe.getUnsafe();
            } catch (SecurityException se) {
                return null;
            }
        }

        static final <K, V> Node<K, V> tabAt(Node<K, V>[] tab, int i) {
            return (Node<K, V>) UNSAFE.getObjectVolatile(tab, ((long) i << ASHIFT) + ABASE);
        }

        static final <K, V> boolean casTabAt(Node<K, V>[] tab, int i, Node<K, V> c, Node<K, V> v) {
            return UNSAFE.compareAndSwapObject(tab, ((long) i << ASHIFT) + ABASE, c, v);
        }

        private static final int ASHIFT;
        private static final int ABASE;

        static {
            try {
                ABASE = UNSAFE.arrayBaseOffset(Node[].class);
                int scale = UNSAFE.arrayIndexScale(Node[].class);
                ASHIFT = 31 - Integer.numberOfLeadingZeros(scale);
            } catch (Exception e) {
                throw new Error(e);
            }
        }

        private final void treeifyBin(Node<K, V>[] tab, int index) {
            // 简化实现
        }

        private final Node<K, V>[] helpTransfer(Node<K, V>[] tab, Node<K, V> f) {
            // 简化实现
            return tab;
        }

        private final void addCount(long x, int check) {
            // 简化实现
        }
    }

    // ConcurrentHashMap的性能特性：
    // 1. 高并发性能好
    // 2. 读操作无锁
    // 3. 写操作使用分段锁或CAS
    // 4. 允许并发读写

    // ConcurrentHashMap的使用场景：
    // 1. 高并发环境
    // 2. 需要线程安全的Map
    // 3. 读多写少的场景
    // 4. 不需要保持顺序
}
```

## 5. 队列接口深度解析

### 5.1 ConcurrentLinkedQueue

```java
import java.util.concurrent.atomic.*;
import java.util.*;

public class ConcurrentLinkedQueueDeepDive {
    // ConcurrentLinkedQueue的简化实现
    private static class MyConcurrentLinkedQueue<E> {
        private static class Node<E> {
            volatile E item;
            volatile Node<E> next;

            Node(E item) {
                UNSAFE.putObject(this, itemOffset, item);
            }

            boolean casItem(E cmp, E val) {
                return UNSAFE.compareAndSwapObject(this, itemOffset, cmp, val);
            }

            void lazySetNext(Node<E> val) {
                UNSAFE.putOrderedObject(this, nextOffset, val);
            }

            boolean casNext(Node<E> cmp, Node<E> val) {
                return UNSAFE.compareAndSwapObject(this, nextOffset, cmp, val);
            }

            private static final sun.misc.Unsafe UNSAFE;
            private static final long itemOffset;
            private static final long nextOffset;

            static {
                try {
                    UNSAFE = sun.misc.Unsafe.getUnsafe();
                    itemOffset = UNSAFE.objectFieldOffset(Node.class.getDeclaredField("item"));
                    nextOffset = UNSAFE.objectFieldOffset(Node.class.getDeclaredField("next"));
                } catch (Exception e) {
                    throw new Error(e);
                }
            }
        }

        private volatile Node<E> head;
        private volatile Node<E> tail;

        public MyConcurrentLinkedQueue() {
            head = tail = new Node<E>(null);
        }

        // 添加元素
        public boolean offer(E e) {
            if (e == null) {
                throw new NullPointerException();
            }

            Node<E> newNode = new Node<E>(e);

            while (true) {
                Node<E> t = tail;
                Node<E> p = t;

                // 找到尾节点
                for (Node<E> q = p.next; q != null; p = q, q = p.next) {
                    t = p;
                }

                // 尝试设置新节点
                if (p.casNext(null, newNode)) {
                    if (p != t) {
                        casTail(t, newNode);
                    }
                    return true;
                }
            }
        }

        // 移除元素
        public E poll() {
            while (true) {
                Node<E> h = head;
                Node<E> p = h;
                Node<E> q = p.next;

                if (q == null) {
                    return null;
                }

                E item = q.item;
                if (q.casItem(item, null)) {
                    if (p.casNext(q, q.next)) {
                        if (p != h) {
                            casHead(h, p);
                        }
                        return item;
                    }
                }
            }
        }

        // 查看元素
        public E peek() {
            Node<E> h = head;
            Node<E> p = h;
            Node<E> q;

            for (q = p.next; q != null; p = q, q = p.next) {
                E item = q.item;
                if (item != null) {
                    return item;
                }
            }

            return null;
        }

        private boolean casTail(Node<E> cmp, Node<E> val) {
            return UNSAFE.compareAndSwapObject(this, tailOffset, cmp, val);
        }

        private boolean casHead(Node<E> cmp, Node<E> val) {
            return UNSAFE.compareAndSwapObject(this, headOffset, cmp, val);
        }

        private static final sun.misc.Unsafe UNSAFE;
        private static final long headOffset;
        private static final long tailOffset;

        static {
            try {
                UNSAFE = sun.misc.Unsafe.getUnsafe();
                headOffset = UNSAFE.objectFieldOffset(MyConcurrentLinkedQueue.class.getDeclaredField("head"));
                tailOffset = UNSAFE.objectFieldOffset(MyConcurrentLinkedQueue.class.getDeclaredField("tail"));
            } catch (Exception e) {
                throw new Error(e);
            }
        }

        public boolean isEmpty() {
            return peek() == null;
        }

        public int size() {
            int count = 0;
            Node<E> p = head.next;
            while (p != null) {
                if (p.item != null) {
                    count++;
                }
                p = p.next;
            }
            return count;
        }
    }

    // ConcurrentLinkedQueue的性能特性：
    // 1. 无锁并发队列
    // 2. 基于CAS操作
    // 3. 高并发性能好
    // 4. 不允许null元素

    // ConcurrentLinkedQueue的使用场景：
    // 1. 高并发环境
    // 2. 需要无锁队列
    // 3. 生产者-消费者模式
    // 4. 事件处理系统
}
```

### 5.2 BlockingQueue实现

```java
import java.util.concurrent.*;
import java.util.*;
import java.util.concurrent.locks.*;

public class BlockingQueueDeepDive {
    // LinkedBlockingQueue的简化实现
    private static class MyLinkedBlockingQueue<E> {
        private static final int DEFAULT_CAPACITY = Integer.MAX_VALUE;
        private final int capacity;

        static class Node<E> {
            E item;
            Node<E> next;

            Node(E x) {
                item = x;
            }
        }

        private final ReentrantLock takeLock = new ReentrantLock();
        private final Condition notEmpty = takeLock.newCondition();

        private final ReentrantLock putLock = new ReentrantLock();
        private final Condition notFull = putLock.newCondition();

        private Node<E> head;
        private Node<E> last;
        private int count;

        public MyLinkedBlockingQueue() {
            this(DEFAULT_CAPACITY);
        }

        public MyLinkedBlockingQueue(int capacity) {
            if (capacity <= 0) {
                throw new IllegalArgumentException();
            }
            this.capacity = capacity;
            last = head = new Node<E>(null);
        }

        // 添加元素
        public void put(E e) throws InterruptedException {
            if (e == null) {
                throw new NullPointerException();
            }

            int c = -1;
            Node<E> node = new Node<E>(e);
            final ReentrantLock putLock = this.putLock;
            final AtomicInteger count = this.count;

            putLock.lockInterruptibly();
            try {
                while (count.get() == capacity) {
                    notFull.await();
                }
                enqueue(node);
                c = count.getAndIncrement();
                if (c + 1 < capacity) {
                    notFull.signal();
                }
            } finally {
                putLock.unlock();
            }

            if (c == 0) {
                signalNotEmpty();
            }
        }

        // 移除元素
        public E take() throws InterruptedException {
            E x;
            int c = -1;
            final AtomicInteger count = this.count;
            final ReentrantLock takeLock = this.takeLock;

            takeLock.lockInterruptibly();
            try {
                while (count.get() == 0) {
                    notEmpty.await();
                }
                x = dequeue();
                c = count.getAndDecrement();
                if (c > 1) {
                    notEmpty.signal();
                }
            } finally {
                takeLock.unlock();
            }

            if (c == capacity) {
                signalNotFull();
            }
            return x;
        }

        // 添加元素到队列
        private void enqueue(Node<E> node) {
            last = last.next = node;
        }

        // 从队列移除元素
        private E dequeue() {
            Node<E> h = head;
            Node<E> first = h.next;
            h.next = h;
            head = first;
            E x = first.item;
            first.item = null;
            return x;
        }

        // 唤醒notEmpty
        private void signalNotEmpty() {
            final ReentrantLock takeLock = this.takeLock;
            takeLock.lock();
            try {
                notEmpty.signal();
            } finally {
                takeLock.unlock();
            }
        }

        // 唤醒notFull
        private void signalNotFull() {
            final ReentrantLock putLock = this.putLock;
            putLock.lock();
            try {
                notFull.signal();
            } finally {
                putLock.unlock();
            }
        }

        public int size() {
            return count.get();
        }

        public boolean isEmpty() {
            return count.get() == 0;
        }

        private final AtomicInteger count = new AtomicInteger(0);
    }

    // ArrayBlockingQueue的简化实现
    private static class MyArrayBlockingQueue<E> {
        private final E[] items;
        private int takeIndex;
        private int putIndex;
        private int count;
        private final ReentrantLock lock;
        private final Condition notEmpty;
        private final Condition notFull;

        @SuppressWarnings("unchecked")
        public MyArrayBlockingQueue(int capacity) {
            if (capacity <= 0) {
                throw new IllegalArgumentException();
            }
            this.items = (E[]) new Object[capacity];
            this.lock = new ReentrantLock(false);
            this.notEmpty = lock.newCondition();
            this.notFull = lock.newCondition();
        }

        // 添加元素
        public void put(E e) throws InterruptedException {
            checkNotNull(e);
            final ReentrantLock lock = this.lock;
            lock.lockInterruptibly();
            try {
                while (count == items.length) {
                    notFull.await();
                }
                enqueue(e);
            } finally {
                lock.unlock();
            }
        }

        // 移除元素
        public E take() throws InterruptedException {
            final ReentrantLock lock = this.lock;
            lock.lockInterruptibly();
            try {
                while (count == 0) {
                    notEmpty.await();
                }
                return dequeue();
            } finally {
                lock.unlock();
            }
        }

        // 添加元素
        private void enqueue(E x) {
            items[putIndex] = x;
            if (++putIndex == items.length) {
                putIndex = 0;
            }
            count++;
            notEmpty.signal();
        }

        // 移除元素
        private E dequeue() {
            final E[] items = this.items;
            E x = items[takeIndex];
            items[takeIndex] = null;
            if (++takeIndex == items.length) {
                takeIndex = 0;
            }
            count--;
            notFull.signal();
            return x;
        }

        private void checkNotNull(E v) {
            if (v == null) {
                throw new NullPointerException();
            }
        }

        public int size() {
            final ReentrantLock lock = this.lock;
            lock.lock();
            try {
                return count;
            } finally {
                lock.unlock();
            }
        }

        public boolean isEmpty() {
            final ReentrantLock lock = this.lock;
            lock.lock();
            try {
                return count == 0;
            } finally {
                lock.unlock();
            }
        }
    }

    // BlockingQueue的使用场景：
    // 1. 生产者-消费者模式
    // 2. 线程池任务队列
    // 3. 消息队列
    // 4. 流量控制

    // 选择BlockingQueue实现的考虑因素：
    // 1. 容量限制
    // 2. 内存使用
    // 3. 性能要求
    // 4. 公平性要求
}
```

## 6. 集合框架的性能优化

### 6.1 初始容量设置

```java
import java.util.*;
import java.util.concurrent.*;

public class CollectionOptimization {
    // 初始容量对性能的影响
    public void initialCapacityImpact() {
        // 不好的做法：使用默认初始容量
        List<Integer> defaultList = new ArrayList<>();
        long startTime = System.nanoTime();
        for (int i = 0; i < 100000; i++) {
            defaultList.add(i);
        }
        long defaultTime = System.nanoTime() - startTime;

        // 好的做法：设置合适的初始容量
        List<Integer> optimizedList = new ArrayList<>(100000);
        startTime = System.nanoTime();
        for (int i = 0; i < 100000; i++) {
            optimizedList.add(i);
        }
        long optimizedTime = System.nanoTime() - startTime;

        System.out.println("Default capacity time: " + defaultTime / 1000000.0 + " ms");
        System.out.println("Optimized capacity time: " + optimizedTime / 1000000.0 + " ms");
        System.out.println("Performance improvement: " + (double) defaultTime / optimizedTime + "x");
    }

    // 集合初始容量选择策略
    public void initialCapacityStrategy() {
        // ArrayList: 如果知道大概大小，设置初始容量
        List<String> names = new ArrayList<>(100);  // 预计100个元素

        // HashMap: 设置合适的初始容量和负载因子
        Map<String, Integer> cache = new HashMap<>(16, 0.75f);

        // HashSet: 类似HashMap
        Set<Integer> uniqueIds = new HashSet<>(1000);

        // ConcurrentHashMap: 考虑并发度和容量
        Map<String, String> concurrentCache = new ConcurrentHashMap<>(64);

        // 集合容量选择原则：
        // 1. 如果知道确切大小，使用确切大小
        // 2. 如果知道大概范围，使用范围的上限
        // 3. 如果不知道大小，使用默认值
        // 4. 考虑负载因子的影响
    }

    // 批量操作优化
    public void bulkOperationOptimization() {
        List<Integer> source = new ArrayList<>(10000);
        for (int i = 0; i < 10000; i++) {
            source.add(i);
        }

        // 不好的做法：逐个添加
        List<Integer> target1 = new ArrayList<>();
        long startTime = System.nanoTime();
        for (Integer item : source) {
            target1.add(item);
        }
        long individualTime = System.nanoTime() - startTime;

        // 好的做法：批量添加
        List<Integer> target2 = new ArrayList<>(source.size());
        startTime = System.nanoTime();
        target2.addAll(source);
        long bulkTime = System.nanoTime() - startTime;

        System.out.println("Individual add time: " + individualTime / 1000000.0 + " ms");
        System.out.println("Bulk add time: " + bulkTime / 1000000.0 + " ms");
        System.out.println("Bulk operation improvement: " + (double) individualTime / bulkTime + "x");
    }
}
```

### 6.2 遍历优化

```java
public class IterationOptimization {
    // 不同遍历方式的性能比较
    public void iterationComparison() {
        List<Integer> list = new ArrayList<>(100000);
        for (int i = 0; i < 100000; i++) {
            list.add(i);
        }

        // 1. 传统for循环
        long startTime = System.nanoTime();
        int sum1 = 0;
        for (int i = 0; i < list.size(); i++) {
            sum1 += list.get(i);
        }
        long forLoopTime = System.nanoTime() - startTime;

        // 2. 增强for循环
        startTime = System.nanoTime();
        int sum2 = 0;
        for (Integer item : list) {
            sum2 += item;
        }
        long enhancedForTime = System.nanoTime() - startTime;

        // 3. 迭代器遍历
        startTime = System.nanoTime();
        int sum3 = 0;
        Iterator<Integer> iterator = list.iterator();
        while (iterator.hasNext()) {
            sum3 += iterator.next();
        }
        long iteratorTime = System.nanoTime() - startTime;

        // 4. Java 8 forEach
        startTime = System.nanoTime();
        final int[] sum4 = {0};
        list.forEach(item -> sum4[0] += item);
        long forEachTime = System.nanoTime() - startTime;

        // 5. Java 8 stream
        startTime = System.nanoTime();
        int sum5 = list.stream().mapToInt(Integer::intValue).sum();
        long streamTime = System.nanoTime() - startTime;

        System.out.println("For loop time: " + forLoopTime / 1000000.0 + " ms");
        System.out.println("Enhanced for loop time: " + enhancedForTime / 1000000.0 + " ms");
        System.out.println("Iterator time: " + iteratorTime / 1000000.0 + " ms");
        System.out.println("forEach time: " + forEachTime / 1000000.0 + " ms");
        System.out.println("Stream time: " + streamTime / 1000000.0 + " ms");
    }

    // ArrayList vs LinkedList遍历性能
    public void arrayListVsLinkedListIteration() {
        List<Integer> arrayList = new ArrayList<>(10000);
        List<Integer> linkedList = new LinkedList<>();

        for (int i = 0; i < 10000; i++) {
            arrayList.add(i);
            linkedList.add(i);
        }

        // 测试ArrayList随机访问
        long startTime = System.nanoTime();
        for (int i = 0; i < arrayList.size(); i++) {
            arrayList.get(i);
        }
        long arrayListRandomAccess = System.nanoTime() - startTime;

        // 测试LinkedList随机访问
        startTime = System.nanoTime();
        for (int i = 0; i < linkedList.size(); i++) {
            linkedList.get(i);
        }
        long linkedListRandomAccess = System.nanoTime() - startTime;

        // 测试ArrayList顺序访问
        startTime = System.nanoTime();
        for (Integer item : arrayList) {
            // 顺序访问
        }
        long arrayListSequentialAccess = System.nanoTime() - startTime;

        // 测试LinkedList顺序访问
        startTime = System.nanoTime();
        for (Integer item : linkedList) {
            // 顺序访问
        }
        long linkedListSequentialAccess = System.nanoTime() - startTime;

        System.out.println("ArrayList random access: " + arrayListRandomAccess / 1000000.0 + " ms");
        System.out.println("LinkedList random access: " + linkedListRandomAccess / 1000000.0 + " ms");
        System.out.println("ArrayList sequential access: " + arrayListSequentialAccess / 1000000.0 + " ms");
        System.out.println("LinkedList sequential access: " + linkedListSequentialAccess / 1000000.0 + " ms");
    }

    // 遍历优化建议
    public void iterationOptimizationTips() {
        List<String> data = new ArrayList<>();

        // 1. 选择合适的遍历方式
        // ArrayList: 随机访问或增强for循环
        for (int i = 0; i < data.size(); i++) {
            // 适合ArrayList
        }

        for (String item : data) {
            // 通用，性能良好
        }

        // 2. 避免在遍历过程中修改集合
        Iterator<String> iterator = data.iterator();
        while (iterator.hasNext()) {
            String item = iterator.next();
            if (item == null) {
                iterator.remove();  // 正确的删除方式
            }
        }

        // 3. 使用Java 8的并行流处理大数据集
        List<Integer> largeData = new ArrayList<>(1000000);
        for (int i = 0; i < 1000000; i++) {
            largeData.add(i);
        }

        int sum = largeData.parallelStream()
                          .mapToInt(Integer::intValue)
                          .sum();

        // 4. 考虑使用forEachOrdered保持顺序
        largeData.parallelStream()
                .forEachOrdered(System.out::println);

        // 5. 使用索引遍历需要位置信息的情况
        for (int i = 0; i < data.size(); i++) {
            if (i % 2 == 0) {
                // 需要索引的操作
            }
        }
    }
}
```

### 6.3 内存优化

```java
public class MemoryOptimization {
    // 使用基本类型集合
    public void primitiveCollections() {
        // 不好的做法：使用包装类型
        List<Integer> integerList = new ArrayList<>();
        integerList.add(1);  // 自动装箱
        integerList.add(2);  // 自动装箱

        // 好的做法：使用专门的基本类型集合库
        // 如 Eclipse Collections或 HPPC
        // List<Integer> optimizedList = IntArrayList.newArrayList();

        // 或者使用数组
        int[] intArray = new int[1000];
        for (int i = 0; i < intArray.length; i++) {
            intArray[i] = i;
        }
    }

    // 避免内存泄漏
    public void avoidMemoryLeaks() {
        // 1. 清理不再使用的集合
        Map<String, Object> cache = new HashMap<>();

        // 使用后清理
        cache.clear();

        // 2. 使用WeakHashMap缓存
        Map<Object, String> weakCache = new WeakHashMap<>();

        // 3. 注意集合的监听器和回调
        List<EventListener> listeners = new ArrayList<>();

        // 提供移除监听器的方法
        public void removeListener(EventListener listener) {
            listeners.remove(listener);
        }

        // 4. 避免静态集合持有对象引用
        // private static final List<Object> GLOBAL_CACHE = new ArrayList<>(); // 危险
    }

    // 选择合适的数据结构
    public void chooseAppropriateDataStructure() {
        // 1. 根据访问模式选择
        // 频繁随机访问：ArrayList
        List<Integer> randomAccessList = new ArrayList<>();

        // 频繁插入删除：LinkedList
        List<Integer> frequentlyModifiedList = new LinkedList<>();

        // 2. 根据数据特性选择
        // 需要唯一性：Set
        Set<String> uniqueNames = new HashSet<>();

        // 需要保持顺序：LinkedHashSet
        Set<String> orderedUniqueNames = new LinkedHashSet<>();

        // 需要排序：TreeSet
        Set<Integer> sortedNumbers = new TreeSet<>();

        // 3. 根据线程安全要求选择
        // 单线程环境：HashMap, ArrayList
        Map<String, String> singleThreadMap = new HashMap<>();

        // 多线程环境：ConcurrentHashMap, CopyOnWriteArrayList
        Map<String, String> multiThreadMap = new ConcurrentHashMap<>();

        // 4. 根据内存敏感度选择
        // 内存敏感：原始数组
        int[] memoryEfficientArray = new int[1000];

        // 功能优先：集合框架
        List<Integer> functionalList = new ArrayList<>();
    }

    // 使用视图和子集合
    public void useViewsAndSubCollections() {
        List<Integer> largeList = new ArrayList<>();
        for (int i = 0; i < 100000; i++) {
            largeList.add(i);
        }

        // 使用子集合而不是复制
        List<Integer> subList = largeList.subList(0, 1000);

        // 使用不可变视图
        List<Integer> unmodifiableList = Collections.unmodifiableList(largeList);

        // 使用同步视图
        List<Integer> synchronizedList = Collections.synchronizedList(largeList);

        // 使用空集合
        List<Integer> emptyList = Collections.emptyList();
        Set<String> emptySet = Collections.emptySet();
        Map<String, String> emptyMap = Collections.emptyMap();

        // 使用单元素集合
        List<Integer> singleList = Collections.singletonList(42);
        Set<String> singleSet = Collections.singleton("hello");
        Map<String, Integer> singleMap = Collections.singletonMap("key", 1);
    }
}
```

## 7. 集合框架的最佳实践

### 7.1 选择集合类型的决策树

```java
public class CollectionSelection {
    // 集合类型选择指南
    public void collectionSelectionGuide() {
        // 需要存储键值对吗？
        //   是 -> 需要线程安全吗？
        //         是 -> ConcurrentHashMap
        //         否 -> 需要保持顺序吗？
        //               是 -> LinkedHashMap
        //               否 -> 需要排序吗？
        //                     是 -> TreeMap
        //                     否 -> HashMap
        //   否 -> 元素需要唯一吗？
        //         是 -> 需要保持顺序吗？
        //               是 -> LinkedHashSet
        //               否 -> 需要排序吗？
        //                     是 -> TreeSet
        //                     否 -> HashSet
        //         否 -> 需要按索引访问吗？
        //               是 -> 需要线程安全吗？
        //                     是 -> CopyOnWriteArrayList
        //                     否 -> 需要频繁在头部/尾部操作吗？
        //                           是 -> LinkedList
        //                           否 -> ArrayList
        //               否 -> 需要队列功能吗？
        //                     是 -> 需要阻塞吗？
        //                           是 -> ArrayBlockingQueue/LinkedBlockingQueue
        //                           否 -> ConcurrentLinkedQueue
        //                     否 -> 需要双端队列吗？
        //                           是 -> ArrayDeque
        //                           否 -> LinkedList
    }

    // 性能考虑因素
    public void performanceConsiderations() {
        // 1. 时间复杂度
        // ArrayList: get/set O(1), add/remove O(n)
        // LinkedList: get/set O(n), add/remove O(1)
        // HashSet: add/remove/contains O(1)
        // HashMap: put/get/remove O(1)
        // TreeSet: add/remove/contains O(log n)
        // TreeMap: put/get/remove O(log n)

        // 2. 空间复杂度
        // ArrayList: 连续内存，空间利用率高
        // LinkedList: 额外空间存储指针
        // HashSet/HashMap: 需要额外空间处理哈希冲突
        // TreeSet/TreeMap: 红黑树结构，空间开销较大

        // 3. 扩容成本
        // ArrayList: 扩容时需要复制数组
        // HashMap: 扩容时需要重新哈希
        // HashSet: 扩容时需要重新哈希
        // LinkedList: 动态扩容，成本低
    }

    // 线程安全考虑
    public void threadSafetyConsiderations() {
        // 1. 同步集合
        List<String> synchronizedList = Collections.synchronizedList(new ArrayList<>());
        Set<String> synchronizedSet = Collections.synchronizedSet(new HashSet<>());
        Map<String, String> synchronizedMap = Collections.synchronizedMap(new HashMap<>());

        // 2. 并发集合
        List<String> concurrentList = new CopyOnWriteArrayList<>();
        Set<String> concurrentSet = new ConcurrentSkipListSet<>();
        Map<String, String> concurrentMap = new ConcurrentHashMap<>();

        // 3. 阻塞队列
        BlockingQueue<String> blockingQueue = new LinkedBlockingQueue<>();

        // 线程安全集合的选择：
        // - 读多写少：CopyOnWriteArrayList, CopyOnWriteArraySet
        // - 高并发读写：ConcurrentHashMap, ConcurrentSkipListMap
        // - 生产者-消费者：BlockingQueue
        // - 需要排序：ConcurrentSkipListSet, ConcurrentSkipListMap
    }
}
```

### 7.2 常见错误和解决方案

```java
public class CommonMistakes {
    // 1. ConcurrentModificationException
    public void concurrentModificationExample() {
        List<String> list = new ArrayList<>();
        list.add("a");
        list.add("b");
        list.add("c");

        // 错误的做法
        try {
            for (String item : list) {
                if (item.equals("b")) {
                    list.remove(item);  // 抛出ConcurrentModificationException
                }
            }
        } catch (ConcurrentModificationException e) {
            System.out.println("Caught ConcurrentModificationException");
        }

        // 正确的做法：使用迭代器
        Iterator<String> iterator = list.iterator();
        while (iterator.hasNext()) {
            String item = iterator.next();
            if (item.equals("b")) {
                iterator.remove();  // 正确的删除方式
            }
        }

        // 正确的做法：使用Java 8的removeIf
        list.removeIf(item -> item.equals("b"));

        // 正确的做法：创建新集合
        List<String> filteredList = new ArrayList<>();
        for (String item : list) {
            if (!item.equals("b")) {
                filteredList.add(item);
            }
        }
    }

    // 2. 空指针异常
    public void nullPointerExceptionExample() {
        // 错误的做法
        List<String> list = null;
        try {
            list.add("hello");  // 抛出NullPointerException
        } catch (NullPointerException e) {
            System.out.println("Caught NullPointerException");
        }

        // 正确的做法：初始化集合
        List<String> safeList = new ArrayList<>();
        safeList.add("hello");

        // 正确的做法：使用空集合
        List<String> emptyList = Collections.emptyList();
        // emptyList.add("hello");  // UnsupportedOperationException

        // 正确的做法：使用Optional
        Optional<List<String>> optionalList = Optional.ofNullable(list);
        List<String> result = optionalList.orElse(Collections.emptyList());
    }

    // 3. 类型安全
    public void typeSafetyExample() {
        // Java 5之前的类型不安全
        List rawList = new ArrayList();
        rawList.add("hello");
        rawList.add(123);  // 编译通过，运行时可能出错

        // 正确的做法：使用泛型
        List<String> typedList = new ArrayList<>();
        typedList.add("hello");
        // typedList.add(123);  // 编译错误

        // 处理遗留代码
        @SuppressWarnings("unchecked")
        List<String> legacyList = (List<String>) rawList;
    }

    // 4. 内存泄漏
    public void memoryLeakExample() {
        // 静态集合可能导致内存泄漏
        // private static final List<Object> STATIC_CACHE = new ArrayList<>();

        // 解决方案：使用WeakHashMap
        Map<Object, String> weakCache = new WeakHashMap<>();

        // 或者在适当的时候清理
        // STATIC_CACHE.clear();

        // 监听器未注销
        List<EventListener> listeners = new ArrayList<>();

        // 提供注销方法
        void addListener(EventListener listener) {
            listeners.add(listener);
        }

        void removeListener(EventListener listener) {
            listeners.remove(listener);
        }
    }

    // 5. 性能陷阱
    public void performanceTrapsExample() {
        // 在循环中创建集合
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

        // 不好的做法
        List<Integer> result = new ArrayList<>();
        for (Integer number : numbers) {
            List<Integer> tempList = new ArrayList<>();  // 不必要的创建
            tempList.add(number);
            result.addAll(tempList);
        }

        // 好的做法
        List<Integer> optimizedResult = new ArrayList<>();
        for (Integer number : numbers) {
            optimizedResult.add(number);
        }

        // 使用LinkedList进行随机访问
        List<Integer> linkedList = new LinkedList<>();
        for (int i = 0; i < 1000; i++) {
            linkedList.add(i);
        }

        // 不好的做法
        for (int i = 0; i < linkedList.size(); i++) {
            linkedList.get(i);  // 性能很差
        }

        // 好的做法
        for (Integer number : linkedList) {
            // 使用增强for循环
        }
    }
}
```

## 8. 集合框架的未来发展

### 8.1 Java 8+的新特性

```java
import java.util.stream.*;
import java.util.*;
import java.util.function.*;

public class ModernCollectionFeatures {
    // 1. Stream API
    public void streamAPIExample() {
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David", "Eve");

        // 过滤和映射
        List<String> filtered = names.stream()
                                  .filter(name -> name.length() > 4)
                                  .map(String::toUpperCase)
                                  .collect(Collectors.toList());

        System.out.println("Filtered names: " + filtered);

        // 聚合操作
        long count = names.stream()
                         .filter(name -> name.startsWith("A"))
                         .count();

        System.out.println("Names starting with A: " + count);

        // 分组
        Map<Integer, List<String>> groupedByLength = names.stream()
                                                         .collect(Collectors.groupingBy(String::length));

        System.out.println("Grouped by length: " + groupedByLength);

        // 并行流
        List<Integer> numbers = new ArrayList<>();
        for (int i = 0; i < 1000000; i++) {
            numbers.add(i);
        }

        long sum = numbers.parallelStream()
                          .mapToInt(Integer::intValue)
                          .sum();

        System.out.println("Parallel sum: " + sum);
    }

    // 2. Lambda表达式
    public void lambdaExpressions() {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

        // 传统方式
        Collections.sort(numbers, new Comparator<Integer>() {
            @Override
            public int compare(Integer a, Integer b) {
                return b.compareTo(a);
            }
        });

        // Lambda方式
        numbers.sort((a, b) -> b.compareTo(a));

        // 方法引用
        numbers.sort(Integer::compareTo);

        // 集合操作
        numbers.forEach(System.out::println);
        numbers.removeIf(n -> n % 2 == 0);
        numbers.replaceAll(n -> n * 2);
    }

    // 3. Optional
    public void optionalExample() {
        List<String> names = Arrays.asList("Alice", null, "Bob", null, "Charlie");

        // 使用Optional处理可能为null的值
        names.stream()
             .filter(Objects::nonNull)
             .map(Optional::ofNullable)
             .forEach(opt -> opt.ifPresent(System.out::println));

        // 使用Optional避免空指针异常
        String result = names.stream()
                            .filter(Objects::nonNull)
                            .findFirst()
                            .orElse("Default");
    }

    // 4. 新的集合工厂方法
    public void collectionFactoryMethods() {
        // Java 9+的集合工厂方法
        List<Integer> immutableList = List.of(1, 2, 3, 4, 5);
        Set<String> immutableSet = Set.of("a", "b", "c");
        Map<String, Integer> immutableMap = Map.of("a", 1, "b", 2, "c", 3);

        // Java 10+的不可变集合拷贝
        List<Integer> copiedList = List.copyOf(immutableList);

        System.out.println("Immutable list: " + immutableList);
        System.out.println("Immutable set: " + immutableSet);
        System.out.println("Immutable map: " + immutableMap);
    }
}
```

### 8.2 未来发展趋势

```java
public class FutureTrends {
    // 1. 更好的并发支持
    public void betterConcurrencySupport() {
        // 虚拟线程对集合的影响
        // 更高效的并发集合
        // 更好的并行算法

        // 示例：使用虚拟线程处理集合
        // ExecutorService virtualExecutor = Executors.newVirtualThreadPerTaskExecutor();
        // virtualExecutor.submit(() -> {
        //     List<String> result = largeCollection.stream()
        //                                         .parallel()
        //                                         .collect(Collectors.toList());
        //     return result;
        // });
    }

    // 2. 内存效率提升
    public void memoryEfficiencyImprovements() {
        // 紧凑集合（Compact Collections）
        // 原始类型集合
        // 更好的内存布局

        // 示例：使用紧凑集合
        // List<Integer> compactList = new CompactArrayList<>();
        // Set<Integer> compactSet = new CompactHashSet<>();
    }

    // 3. 函数式编程增强
    public void functionalProgrammingEnhancement() {
        // 更多的函数式操作
        // 更好的惰性求值
        // 更丰富的流操作

        // 示例：增强的流操作
        // List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
        // numbers.stream()
        //        .takeWhile(n -> n < 4)
        //        .dropWhile(n -> n < 2)
        //        .forEach(System.out::println);
    }

    // 4. 性能监控和优化
    public void performanceMonitoring() {
        // 内置性能监控
        // 自适应优化
        // 更好的诊断工具

        // 示例：性能监控
        // List<Integer> monitoredList = new MonitoredArrayList<>();
        // monitoredList.setPerformanceListener(new PerformanceListener() {
        //     @Override
        //     public void onOperation(String operation, long duration) {
        //         System.out.println("Operation " + operation + " took " + duration + "ns");
        //     }
        // });
    }
}
```

## 9. 总结与最佳实践

### 9.1 集合框架设计哲学

```java
public class DesignPhilosophy {
    // Java集合框架的设计原则
    public void designPrinciples() {
        // 1. 接口与实现分离
        // 面向接口编程，而不是面向实现编程
        List<String> list = new ArrayList<>();  // 可以随时切换实现

        // 2. 算法与数据结构分离
        // Collections工具类提供算法，集合提供数据结构
        List<Integer> numbers = Arrays.asList(3, 1, 4, 1, 5);
        Collections.sort(numbers);  // 算法与数据结构分离

        // 3. 统一的迭代方式
        // 所有集合都支持相同的遍历方式
        for (String item : list) {
            System.out.println(item);
        }

        // 4. 灵活的扩展机制
        // 通过继承和组合可以轻松扩展集合功能
        class CustomList<E> extends ArrayList<E> {
            public void customOperation() {
                // 自定义操作
            }
        }
    }

    // 性能与可读性的平衡
    public void performanceVsReadability() {
        // 选择集合时要考虑：
        // 1. 功能需求
        // 2. 性能要求
        // 3. 代码可读性
        // 4. 维护成本

        // 示例：简单的场景使用简单实现
        List<String> simpleList = new ArrayList<>();  // 简单易读

        // 示例：复杂场景使用专门实现
        Map<String, Object> cache = new ConcurrentHashMap<>();  // 高性能需求

        // 示例：平衡的选择
        Set<String> uniqueNames = new HashSet<>();  // 功能明确，性能良好
    }
}
```

### 9.2 最佳实践总结

```java
public class BestPracticesSummary {
    // 集合选择的最佳实践
    public void collectionSelectionBestPractices() {
        // 1. 了解每种集合的特性
        //    - ArrayList: 随机访问快，插入删除慢
        //    - LinkedList: 随机访问慢，插入删除快
        //    - HashSet: 快速查找，无序
        //    - TreeSet: 有序，查找速度较慢
        //    - HashMap: 快速查找，允许null键值
        //    - TreeMap: 有序，查找速度较慢

        // 2. 根据实际需求选择
        //    - 需要随机访问：ArrayList
        //    - 需要快速查找：HashSet, HashMap
        //    - 需要保持顺序：LinkedHashSet, LinkedHashMap
        //    - 需要排序：TreeSet, TreeMap
        //    - 需要线程安全：ConcurrentHashMap, CopyOnWriteArrayList

        // 3. 考虑性能优化
        //    - 设置合适的初始容量
        //    - 选择合适的遍历方式
        //    - 避免不必要的集合复制
        //    - 使用批量操作
    }

    // 代码质量的最佳实践
    public void codeQualityBestPractices() {
        // 1. 使用泛型保证类型安全
        List<String> typedList = new ArrayList<>();

        // 2. 使用接口而不是具体类
        List<String> interfaceList = new ArrayList<>();

        // 3. 避免使用原始类型
        // List list = new ArrayList();  // 不要这样做

        // 4. 正确处理并发修改
        Iterator<String> iterator = list.iterator();
        while (iterator.hasNext()) {
            String item = iterator.next();
            if (item == null) {
                iterator.remove();
            }
        }

        // 5. 使用集合工具类
        Collections.emptyList();
        Collections.singletonList("item");
        Collections.singletonMap("key", "value");

        // 6. 注意内存管理
        // 及时清理不再使用的集合引用
        // 使用WeakHashMap缓存
        // 避免静态集合持有对象引用
    }

    // 调试和测试的最佳实践
    public void debuggingAndTesting() {
        // 1. 使用断言验证集合状态
        List<String> list = new ArrayList<>();
        list.add("item");
        assert list.size() == 1 : "List should contain one element";

        // 2. 使用日志记录集合操作
        System.out.println("List size: " + list.size());

        // 3. 编写单元测试
        // @Test
        // public void testListOperations() {
        //     List<String> list = new ArrayList<>();
        //     list.add("test");
        //     assertEquals(1, list.size());
        // }

        // 4. 使用性能分析工具
        // - VisualVM
        // - JProfiler
        // - YourKit
    }
}
```

## 结语

Java集合框架是Java程序设计的基石，它体现了Java语言的设计哲学和工程智慧。通过本文的学习，你应该掌握了：

1. 集合框架的设计思想和体系结构
2. 各种集合类的内部实现原理
3. 集合框架的性能特性和优化策略
4. 线程安全集合的使用
5. 集合操作的最佳实践

记住，选择合适的集合类型是写出高效代码的关键。**优秀的Java开发者不仅要了解如何使用集合，更要理解其背后的设计原理和性能特性。**

随着Java的发展，集合框架也在不断演进。Java 8的Stream API、Lambda表达式，以及未来的虚拟线程和函数式编程增强，都将为集合框架带来新的可能性。保持学习，跟上技术发展的步伐，才能在Java开发的道路上不断进步。

**集合框架不仅是一组工具，更是一种思维方式和设计哲学。理解这种哲学，才能写出真正优雅、高效的Java代码。**

---

*这篇文章深入剖析了Java集合框架的各个方面，从设计哲学到实现原理，从性能优化到最佳实践。通过大量的代码示例和实际案例分析，帮助读者全面理解Java集合框架的精髓。浅者可以学会基本的集合使用，深者能够理解其内部原理和性能优化策略。*