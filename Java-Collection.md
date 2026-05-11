# Java-Collection

Java 集合框架

<div align="center"> <img src="https://oss.javaguide.cn/github/javaguide/java/collection/java-collection-hierarchy.png" width="100%"/> </div>



## 基础概念

### Java线程安全集合

| 类别                                | 类名                    | 核心特点           | 实现/加锁机制简述                                            |
| :---------------------------------- | :---------------------- | :----------------- | :----------------------------------------------------------- |
| **传统集合** (java.util)            | `Vector`                | 动态数组，线程安全 | 方法用 `synchronized` 修饰，锁整个对象，有额外开销           |
|                                     | `Hashtable`             | 哈希表，线程安全   | 方法用 `synchronized` 修饰，锁整个表，不支持 null 键/值，性能较差 |
| **并发集合** (java.util.concurrent) | `ConcurrentHashMap`     | 高效并发哈希表     | JDK1.7分段锁，JDK1.8桶粒度锁 + CAS + 红黑树                  |
|                                     | `ConcurrentSkipListMap` | 可排序的并发Map    | 基于跳表算法，对数时间复杂度                                 |
|                                     | `ConcurrentSkipListSet` | 有序的并发Set      | 底层基于 `ConcurrentSkipListMap`                             |
|                                     | `CopyOnWriteArraySet`   | 无序的并发Set      | 底层基于 `CopyOnWriteArrayList`，写时复制                    |
|                                     | `CopyOnWriteArrayList`  | 并发的动态数组     | 写操作复制新数组（加锁），读操作无锁                         |
|                                     | `ConcurrentLinkedQueue` | 高性能并发队列     | 无锁，基于CAS，非阻塞                                        |
|                                     | `BlockingQueue` (接口)  | 支持阻塞的队列     | 读写可能阻塞（如满/空时），简化线程间数据共享                |
|                                     | `LinkedBlockingDeque`   | 线程安全双端队列   | 链表结构，读写未分离锁，支持阻塞                             |
|                                     | `ConcurrentLinkedDeque` | 并发双端队列       | 基于链接节点的无锁并发双端队列，非阻塞                       |



## ArrayList

### ArrayList 和 Array（数组）的区别

`ArrayList` 内部基于动态数组实现，比 `Array`（静态数组） 使用起来更加灵活：

- `ArrayList`会根据实际存储的元素动态地扩容或缩容，而 `Array` 被创建之后就不能改变它的长度了。
- `ArrayList` 允许你使用泛型来确保类型安全，`Array` 则不可以。
- `ArrayList` 中只能存储对象。对于基本类型数据，需要使用其对应的包装类（如 Integer、Double 等）。`Array` 可以直接存储基本类型数据，也可以存储对象。
- `ArrayList` 支持插入、删除、遍历等常见操作，并且提供了丰富的 API 操作方法，比如 `add()`、`remove()`等。`Array` 只是一个固定长度的数组，只能按照下标访问其中的元素，不具备动态添加、删除元素的能力。
- `ArrayList`创建时不需要指定大小，而`Array`创建时必须指定大小。



### ArrayList 插入和删除元素的时间复杂度

对于插入：

- 头部插入：由于需要将所有元素都依次向后移动一个位置，因此时间复杂度是 O(n)。
- 尾部插入：当 `ArrayList` 的容量未达到极限时，往列表末尾插入元素的时间复杂度是 O(1)，因为它只需要在数组末尾添加一个元素即可；当容量已达到极限并且需要扩容时，则需要执行一次 O(n) 的操作将原数组复制到新的更大的数组中，然后再执行 O(1) 的操作添加元素。
- 指定位置插入：需要将目标位置之后的所有元素都向后移动一个位置，然后再把新元素放入指定位置。这个过程需要移动平均 n/2 个元素，因此时间复杂度为 O(n)。

对于删除：

- 头部删除：由于需要将所有元素依次向前移动一个位置，因此时间复杂度是 O(n)。
- 尾部删除：当删除的元素位于列表末尾时，时间复杂度为 O(1)。
- 指定位置删除：需要将目标元素之后的所有元素向前移动一个位置以填补被删除的空白位置，因此需要移动平均 n/2 个元素，时间复杂度为 O(n)。



### ArrayList 与 Vector 区别

| 对比维度       | `ArrayList`                                                  | `Vector`                                                     |
| :------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **线程安全性** | 非线程安全，无内置同步机制，多线程并发修改可能抛出 `ConcurrentModificationException` | 线程安全，大部分方法（如 `add`、`remove`、`get`）用 `synchronized` 修饰 |
| **性能**       | 单线程或自行保证线程安全时性能更优，无加锁开销               | 有加锁/释放锁开销，单线程环境下效率通常低于 `ArrayList`      |
| **扩容机制**   | 扩容为原来的 **1.5 倍**（`oldCapacity + (oldCapacity >> 1)`）（JDK 1.6 起沿用） | 默认扩容为原来的 **2 倍**。支持通过构造方法指定增长因子，可灵活控制扩容幅度 |
| **扩容灵活性** | 不支持自定义增长因子                                         | 可通过构造方法指定增长因子，灵活控制每次扩容的大小           |
| **使用建议**   | 优先使用，性能更好；需要线程安全时，可用 `Collections.synchronizedList()` 包装 | 不推荐在新代码中使用；除非需要内置线程安全且无法使用 `CopyOnWriteArrayList` 或 `Collections.synchronizedList()` |



### ArrayList 与 LinkedList 区别

- **是否保证线程安全：** `ArrayList` 和 `LinkedList` 都是不同步的，也就是不保证线程安全；
- **底层数据结构：** `ArrayList` 底层使用的是 **`Object` 数组**；`LinkedList` 底层使用的是 **双向链表** 数据结构（JDK1.6 之前为循环链表，JDK1.7 取消了循环。注意双向链表和双向循环链表的区别，下面有介绍到！）
- 插入和删除是否受元素位置的影响：
  - `ArrayList` 采用数组存储，所以插入和删除元素的时间复杂度受元素位置的影响。 比如：执行`add(E e)`方法的时候， `ArrayList` 会默认在将指定的元素追加到此列表的末尾，这种情况时间复杂度就是 O(1)。但是如果要在指定位置 i 插入和删除元素的话（`add(int index, E element)`），时间复杂度就为 O(n)。因为在进行上述操作的时候集合中第 i 和第 i 个元素之后的(n-i)个元素都要执行向后位/向前移一位的操作。
  - `LinkedList` 采用链表存储，所以在头尾插入或者删除元素不受元素位置的影响（`add(E e)`、`addFirst(E e)`、`addLast(E e)`、`removeFirst()`、 `removeLast()`），时间复杂度为 O(1)，如果是要在指定位置 `i` 插入和删除元素的话（`add(int index, E element)`，`remove(Object o)`,`remove(int index)`）， 时间复杂度为 O(n) ，因为需要先移动到指定位置再插入和删除。
- **是否支持快速随机访问：** `LinkedList` 不支持高效的随机元素访问，而 `ArrayList`（实现了 `RandomAccess` 接口） 支持。快速随机访问就是通过元素的序号快速获取元素对象(对应于`get(int index)`方法)。
- **内存空间占用：** `ArrayList` 的空间浪费主要体现在在 list 列表的结尾会预留一定的容量空间，而 LinkedList 的空间花费则体现在它的每一个元素都需要消耗比 ArrayList 更多的空间（因为要存放直接后继和直接前驱以及数据）。



### 将ArrayList变成线程安全有哪些方法

不是线程安全的，ArrayList变成线程安全的方式有：

- 使用Collections类的synchronizedList方法将ArrayList包装成线程安全的List：

```java
List<String> synchronizedList = Collections.synchronizedList(arrayList);
```

- 使用CopyOnWriteArrayList类代替ArrayList，它是一个线程安全的List实现：

```java
CopyOnWriteArrayList<String> copyOnWriteArrayList = new CopyOnWriteArrayList<>(arrayList);
```

- 使用Vector类代替ArrayList，Vector是线程安全的List实现：

```java
Vector<String> vector = new Vector<>(arrayList);
```



### ArrayList 线程不安全的三个表现

1. **部分值为 `null`**
2. **索引越界异常**
3. **实际 `size` 与添加元素数量不符**

核心原因分析（以 `add` 方法为例）

```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // 判断是否需要扩容
    elementData[size++] = e;           // 赋值 + size自增
    return true;
}
```

其中 `elementData[size++] = e;` 非原子操作，包含：

- 在 `size` 位置赋值
- `size` 自增

| 问题            | 发生过程简述                                                 |
| :-------------- | :----------------------------------------------------------- |
| **值为 `null`** | 线程1、2 都判断无需扩容；线程1 在索引 `size` 处赋值后未自增，线程2 在同一位置再次赋值；两个线程先后执行 `size++`，导致某索引位置未被赋值（为 `null`） |
| **索引越界**    | 线程1、2 都判断无需扩容；线程1 赋值后执行 `size++`；线程2 此时拿到的 `size` 已变大，再赋值时索引超出数组容量 |
| **`size` 不符** | `size++` 不是原子操作（读取→加1→写回）；多线程同时执行时，可能基于相同的旧 `size` 加1并覆盖，导致实际自增次数少于添加次数 |



### ArrayList的扩容机制

**触发条件**

当 `add` 方法添加元素时，若 `size + 1` 超过当前内部数组的 `elementData.length`，则触发扩容。

**扩容核心步骤**

1. **计算新容量**
   - 新容量 = 旧容量 + 旧容量右移1位（即 `oldCapacity + (oldCapacity >> 1)`，相当于 **1.5 倍**）
   - 若新容量仍小于 `minCapacity`（所需最小容量），则使用 `minCapacity`
   - 若新容量超过最大数组限制（`Integer.MAX_VALUE - 8`），则进行大容量处理
2. **创建新数组**
   根据新容量创建新的对象数组
3. **复制元素**
   使用 `System.arraycopy` 将原数组元素复制到新数组
4. **更新引用**
   将 ArrayList 内部的 `elementData` 引用指向新数组



### CopyOnWriteArrayList是如何实现线程安全的

CopyOnWriteArrayList 通过“写时复制 + 互斥锁 + volatile 数组引用”保证线程安全：写操作加锁复制一份新数组进行修改，最后用 `volatile` 替换引用；读操作直接读旧数组，无需加锁。

```java
private transient volatile Object[] array;
```

在写入操作时，加了一把互斥锁ReentrantLock以保证线程安全。

```java
public boolean add(E e) {
    //获取锁
    final ReentrantLock lock = this.lock;
    //加锁
    lock.lock();
    try {
        //获取到当前List集合保存数据的数组
        Object[] elements = getArray();
        //获取该数组的长度（这是一个伏笔，同时len也是新数组的最后一个元素的索引值）
        int len = elements.length;
        //将当前数组拷贝一份的同时，让其长度加1
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        //将加入的元素放在新数组最后一位，len不是旧数组长度吗，为什么现在用它当成新数组的最后一个元素的下标？建议自行画图推演，就很容易理解。
        newElements[len] = e;
        //替换引用，将数组的引用指向给新数组的地址
        setArray(newElements);
        return true;
    } finally {
        //释放锁
        lock.unlock();
    }
}
```

**核心步骤：**

1. 加 `ReentrantLock` 互斥锁
2. 将原数组拷贝一份，长度 +1
3. 新元素放入新数组末尾
4. 用新数组替换原数组引用（`volatile` 保证可见性）
5. 释放锁

> ⚠️ 写操作期间，其他写线程会被阻塞，但**读线程完全不受影响**（读的是旧数组）

读操作是没有加锁的

```java
public E get(int index) {
    return get(getArray(), index);
}
```



## Set

### Comparable 和 Comparator 的区别

| 对比维度          | `Comparable`                                      | `Comparator`                               |
| :---------------- | :------------------------------------------------ | :----------------------------------------- |
| **包路径**        | `java.lang`                                       | `java.util`                                |
| **核心方法**      | `compareTo(Object obj)`                           | `compare(Object o1, Object o2)`            |
| **排序位置**      | **在类的内部**实现                                | **在类的外部**实现                         |
| **侵入性**        | 侵入性强，修改类本身                              | 侵入性弱，不修改原类                       |
| **单一 / 多排序** | 一个类只能有一种自然排序                          | 一个类可以配合多个 Comparator 实现多种排序 |
| **使用场景**      | 当类有“默认/自然顺序”时（如 `String`、`Integer`） | 当需要多种排序规则，或无法修改原类源码时   |
| **调用方式**      | `Collections.sort(list)`                          | `Collections.sort(list, comparator)`       |

**`Comparable` 是“我默认怎么排”；`Comparator` 是“你想让我怎么排”。一个类只能有一个默认排序，但可以有无数种外部排序规则。**





## HashMap

**JDK 1.7 vs 1.8 核心对比**

| 对比项       | JDK 1.7                    | JDK 1.8                      |
| :----------- | :------------------------- | :--------------------------- |
| 数据结构     | 数组 + 链表                | 数组 + 链表 / 红黑树         |
| 插入方式     | 头插法                     | 尾插法                       |
| 扰动次数     | 4 次                       | 1 次（高 16 位异或低 16 位） |
| 哈希碰撞优化 | 链表拉链                   | 链表 → 红黑树（长链表时）    |
| 扩容后顺序   | 链表顺序反转（可能死循环） | 顺序不变（无死循环问题）     |

**扰动函数（JDK 1.7）**

```java
static int hash(int h) {
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

- 对 `hashCode` 进行 **4 次扰动**
- 目的：让高低位 bits 参与运算，减少哈希碰撞
- 缺点：扰动次数较多，性能略低

**扰动函数（JDK 1.8 简化版）**

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

- **仅扰动 1 次**：高 16 位与低 16 位异或
- 性能比 JDK 1.7 更好，同时保持较好的分布性

**树化条件（两个条件同时满足）**

| 条件                                         | 说明               |
| :------------------------------------------- | :----------------- |
| 链表长度 ≥ `TREEIFY_THRESHOLD`（默认 8）     | 单个桶位链表过长   |
| 数组长度 ≥ `MIN_TREEIFY_CAPACITY`（默认 64） | 避免数组过小时树化 |

> 若链表长度 ≥ 8 但数组长度 < 64 → **优先扩容数组**，而非树化

**为什么选择阈值 8 和 64？**

1. 泊松分布表明，链表长度达到 8 的概率极低（小于千万分之一）。在绝大多数情况下，链表长度都不会超过 8。阈值设置为 8，可以保证性能和空间效率的平衡。
2. 数组长度阈值 64 同样是经过实践验证的经验值。在小数组中扩容成本低，优先扩容可以避免过早引入红黑树。数组大小达到 64 时，冲突概率较高，此时红黑树的性能优势开始显现。

 **`HashMap` 的长度是 2 的幂次方的原因**

1. 位运算效率更高：位运算(&)比取余运算(%)更高效。当长度为 2 的幂次方时，`hash % length` 等价于 `hash & (length - 1)`。
2. 可以更好地保证哈希值的均匀分布：扩容之后，在旧数组元素 hash 值比较均匀的情况下，新数组元素也会被分配的比较均匀，最好的情况是会有一半在新数组的前半部分，一半在新数组后半部分。
3. 扩容机制变得简单和高效：扩容后只需检查哈希值高位的变化来决定元素的新位置，要么位置不变（高位为 0），要么就是移动到新位置（高位为 1，原索引位置+原容量）。

**`get`方法**

`get` 方法通过 key 的 hash 值定位到数组桶位，先检查第一个节点，若匹配则返回；否则根据后续节点是链表还是红黑树，分别遍历或树查找，找到则返回 value，否则返回 null。

**`put`方法**

1. 计算 hash 并定位数组下标
2. 若桶位为空 → 直接插入
3. 若桶位不为空 → 检查第一个节点是否 key 相同
4. 若 key 相同 → 覆盖 value
5. 若 key 不同 → 遍历链表或红黑树查找
6. 找到相同 key → 覆盖；未找到 → 追加新节点
7. 链表长度超过阈值（8）且数组长度 ≥ 64 → 转红黑树
8. 元素总数超过扩容阈值（size > 容量 × 0.75）→ 扩容为 2 倍
9. 完成操作



### 为什么 HashMap 不选 AVL 树，而是红黑树

1. 树化是低频场景，不值得为严格平衡付出高开销

- HashMap 只在 **链表长度 ≥ 8 且数组 ≥ 64** 时才转树
- 树化本身是为了解决“极端哈希冲突”，是兜底手段，不是核心操作
- AVL 树的频繁旋转在低频场景下是性能浪费

2. HashMap 增删频率不低，需要平衡增删 + 查找的综合性能

- HashMap 不是只读结构，`put` 和 `remove` 操作非常频繁
- 红黑树旋转次数少 → 插入/删除更高效
- AVL 树虽然查找稍快，但增删开销过大，得不偿失

3. 红黑树的 O(log n) 已经足够

- 链表长度达到 8 时才转树 → 树节点规模通常不大
- 即使红黑树高度略高于 AVL 树，`O(log n)` 的差异在几百节点内几乎不可感知
- 工程上更看重实际执行效率，而非理论最优



### HashMap是线程安全的吗？

hashmap不是线程安全的，hashmap在多线程会存在下面的问题：

- JDK 1.7 HashMap 采用数组 + 链表 + 头插法。并发扩容时，多个线程同时 transfer 可能形成环形链表，导致 `get` 操作陷入死循环；同时也会发生数据丢失。
- JDK 1.8 HashMap 改为数组 + 链表/红黑树 + 尾插法，扩容时保持节点顺序，解决了死循环问题。但由于 `put` 操作仍然不是原子的（既没有使用 CAS，也没有使用任何锁（如 `synchronized`、`ReentrantLock`）），多线程环境下仍会出现数据覆盖（如两个线程同时写入同一数组桶位，后覆盖前）

如果要保证线程安全，可以通过这些方法来保证：

- 多线程环境可以使用Collections.synchronizedMap同步加锁的方式，还可以使用HashTable，但是同步的方式显然性能不达标，而ConcurrentHashMap更适合高并发场景使用。
- ConcurrentHashMap在JDK1.7和1.8的版本改动比较大，1.7使用Segment+HashEntry分段锁的方式实现，1.8则抛弃了Segment，改为使用CAS+synchronized+Node实现，同样也加入了红黑树，避免链表过长导致性能的问题。



### 为什么HashMap一般使用String做Key

用 string 做 key，因为 String对象是不可变的，一旦创建就不能被修改，这确保了Key的稳定性。如果Key是可变的，可能会导致hashCode和equals方法的不一致，进而影响HashMap的正确性。

hashmap key可以为null

- hashMap中使用hash()方法来计算key的哈希值，当key为空时，直接另key的哈希值为0，不走key.hashCode()方法
- hashMap虽然支持key和value为null，但是null作为key只能有一个，null作为value可以有多个；
- 因为hashMap中，如果key值一样，那么会覆盖相同key值的value为最新，所以key为null只能有一个。



### HashMap的扩容机制

hashMap默认的负载因子是0.75，即如果hashmap中的元素个数超过了总容量75%，则会触发扩容，扩容分为两个步骤：

- **第1步**是对哈希表长度的扩展（2倍）
- **第2步**是将旧哈希表中的数据放到新的哈希表中。

HashMap 用的索引计算公式是：

```java
index = hash & (length - 1) // 计算数组下标
hash & oldCap == 0 // 原位置
hash & oldCap != 0 // 原位置 + oldCap
```

因为使用的是2次幂的扩展(指长度扩为原来2倍)。所以，元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置。



### HashMap vs Hashtable

| 对比维度      | HashMap                                             | Hashtable                                              |
| :------------ | :-------------------------------------------------- | :----------------------------------------------------- |
| **线程安全**  | ❌ 不安全（效率高）                                  | ✅ 安全（方法用 `synchronized` 修饰，效率低）           |
| **null 支持** | ✅ 允许： • null key 最多 1 个 • null value 可以多个 | ❌ 不允许：null key/value 会抛 NPE                      |
| **默认容量**  | 16                                                  | 11                                                     |
| **扩容机制**  | 变为原来 **2 倍**                                   | 变为原来 **2n + 1**                                    |
| **容量指定**  | 若给定初始容量，会扩充为 **2 的幂次方**             | 直接使用给定大小                                       |
| **底层结构**  | 数组 + 链表 / 红黑树（JDK 1.8+）                    | 数组 + 链表                                            |
| **树化条件**  | 链表长度 > 8 且数组长度 ≥ 64 时转红黑树             | 无红黑树                                               |
| **现状**      | 优先使用                                            | 已淘汰，不推荐使用；线程安全场景用 `ConcurrentHashMap` |



## ConcurrentHashMap

| 对比维度         | JDK 1.7                                    | JDK 1.8                                |
| :--------------- | :----------------------------------------- | :------------------------------------- |
| **数据结构**     | 数组 + 链表                                | 数组 + 链表 / 红黑树                   |
| **锁机制**       | 分段锁（`Segment` 继承 `ReentrantLock`）   | `CAS` + `synchronized`（锁桶位头节点） |
| **锁粒度**       | 较大（一个 Segment 锁一个 HashEntry 数组） | 更小（锁单个桶位头节点）               |
| **并发度**       | 理论最大并发数 = Segment 数组长度          | 理论最大并发数 = 数组长度              |
| **查询效率**     | O(n)（链表遍历）                           | O(log n)（红黑树优化）                 |
| **线程安全方式** | 每个 Segment 独立加锁                      | `volatile` + `CAS` + `synchronized`    |

JDK 1.7 ConcurrentHashMap中的分段锁是用了 ReentrantLock，是一个可重入的锁。



### 为什么有了 synchronized 还要用 CAS？

核心原因：根据不同场景的锁竞争程度，选择最合适的线程安全手段，是一种性能权衡。

| 场景                           | 使用手段                 | 原因                                                         |
| :----------------------------- | :----------------------- | :----------------------------------------------------------- |
| **桶位为空**（无哈希冲突）     | `CAS`（乐观锁）          | 哈希碰撞概率低，用 CAS 通过少量自旋即可完成，避免加锁开销    |
| **桶位不为空**（发生哈希冲突） | `synchronized`（悲观锁） | 冲突意味着容量紧张或线程竞争激烈，此时悲观锁比 CAS 的反复自旋更高效 |

无竞争或低竞争时用 CAS（轻量），高竞争时用 synchronized（稳定）。



### **ConcurrentHashMap 用了乐观锁还是悲观锁？**

两者都用，根据不同操作阶段选择不同策略。

| 操作阶段                    | 锁类型          | 具体手段                                 |
| :-------------------------- | :-------------- | :--------------------------------------- |
| **初始化数组**              | 乐观锁          | `volatile` + `CAS`                       |
| **桶位为 null**（无冲突）   | 乐观锁          | `CAS` 直接设置节点                       |
| **桶位不为 null**（有冲突） | 悲观锁          | `synchronized` 锁头节点，遍历链表/红黑树 |
| **扩容协助**                | 乐观锁 + 悲观锁 | `CAS` 领取任务 + `synchronized` 迁移数据 |





