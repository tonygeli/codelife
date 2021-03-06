# 太棒了！HashMap和 ConcurrentHashMap的问题终于总结清楚了

# 一、什么是哈希表

在讨论哈希表之前，我们先大概了解下其他数据结构在新增，查找等基础操作执行性能

**数组**

采用一段连续的存储单元来存储数据。对于指定下标的查找，时间复杂度为O(1)；

通过给定值进行查找，需要遍历数组，逐一比对给定关键字和数组元素，时间复杂度为O(n)，当然，对于有序数组，则可采用二分查找，插值查找，斐波那契查找等方式，可将查找复杂度提高为O(logn)；

对于一般的插入删除操作，涉及到数组元素的移动，其平均复杂度也为O(n)

**线性链表**

对于链表的新增，删除等操作（在找到指定操作位置后），仅需处理结点间的引用即可，时间复杂度为O(1)，而查找操作需要遍历链表逐一进行比对，复杂度为O(n)

**二叉树**

对一棵相对平衡的有序二叉树，对其进行插入，查找，删除等操作，平均复杂度均为O(logn)。

**数组**

相比上述几种数据结构，在哈希表中进行添加，删除，查找等操作，性能十分之高，不考虑哈希冲突的情况下，仅需一次定位即可完成，时间复杂度为O(1).

哈希表上面的特性，哈希表的主干就是数组。

![太棒了！HashMap和 ConcurrentHashMap的问题终于总结清楚了](../../images/b2383bced1804e81b1165b33253dba3f)



比如我们要新增或查找某个元素，我们通过把当前元素的关键字 通过某个函数映射到数组中的某个位置，通过数组下标一次定位就可完成操作。

**存储位置 = f(关键字)**

其中，这个函数f一般称为哈希函数，这个函数的设计好坏会直接影响到哈希表的优劣。查找操作同理，先通过哈希函数计算出实际存储地址，然后从数组中对应地址取出即可。

# 二、哈希冲突

通过哈希函数得出的实际存储地址相同怎么办？也就是说，**当我们对某个元素进行哈希运算，得到一个存储地址，然后要进行插入的时候，发现已经被其他元素占用了**，其实这就是所谓的哈希冲突，也叫哈希碰撞。为什么要重写 hashcode 和 equals 方法？这篇也推荐看下。

> 哈希函数的设计至关重要，好的哈希函数会尽可能地保证 计算简单和散列地址分布均匀,但是不可能设计出一个绝对完美的哈希函数，我们需要清楚的是，数组是一块连续的固定长度的内存空间，再好的哈希函数也不能保证得到的存储地址绝对不发生冲突。

哈希冲突的解决方案有多种:开放定址法（发生冲突，继续寻找下一块未被占用的存储地址），再散列函数法，链地址法，HashMap即是采用了链地址法.

- JDK7 使用了数组+链表的方式
- JDK8 使用了数组+链表+红黑树的方式

# 三、HashMap的实现原理

HashMap的主干是一个Entry数组。Entry是HashMap的基本组成单元，每一个Entry包含一个key-value键值对。

> transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;

Entry是HashMap中的一个静态内部类。

```java
static class Entry<K,V> implements Map.Entry<K,V> {
	final K key;
  V value;
  Entry<K, V> next;		// 存储指向下一个Entry的引用，单链表结构
  int hash;						// 对key的hashcode值进行hash运算后得到的值，存储在Entry避免重复计算
  
  Entry (int h, K k, V v, Entry<K, V> n) {
    value = v;next = n;key = k;hash = h;
  }
}
```

 

HashMap的整体结构如下:

![太棒了！HashMap和 ConcurrentHashMap的问题终于总结清楚了](../../images/b2799b2c4a024e279eb350fe2dbf6b6b.png)



- **解决冲突的链表的长度影响到HashMap查询的效率**
  简单来说，HashMap由数组+链表组成的，数组是HashMap的主体，链表则是主要为了解决哈希冲突而存在的，如果定位到的数组位置不含链表（当前entry的next指向null）,那么对于查找，添加等操作很快，仅需一次寻址即可；如果定位到的数组包含链表，对于添加操作，其时间复杂度为O(n)，首先遍历链表，存在即覆盖，否则新增；对于查找操作来讲，仍需遍历链表，然后通过key对象的equals方法逐一比对查找。所以，性能考虑，HashMap中的链表出现越少，性能才会越好。
- **发生冲突关于entry节点插入链表还是链头呢？**
  JDK7:插入链表的头部，头插法JDK8:插入链表的尾部，尾插法

### JDK7

```java
public V put(K key, V value) {
	if (key == null)
		return putForNullKey(value);
  int hash = hash(key.hashCode());
  int i = indexFor(hash, table.length);
  for (Entry<K,V> e = table[i]; e != null; e = e.next) {
    Object k;
    if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
      V oldValue = e.value;
      e.value = value;
      e.recordAccess(this);
      return oldValue;
    }
  }
  
  modCount++;
  addEntry(hash, key, value, i);
  return null;
}
```

阅读源码发现，如果遍历链表都没法发现相应的key值的话，则会调用addEntry方法在链表添加一个Entry，重点就在与addEntry方法是如何插入链表的，addEntry方法源码如下：

```java
void addEntry(int hash, K key, V value; int bucketIndex) {
	Entry<K,V> e = table[bucketIndex];
  table[bucketIndex] = new Entry<K,V>(hash, key, value, e);
  if (size++ >= threshold)
    return (2 * table.length);
}
```

这里构造了一个新的Entry对象（构造方法的最后一个参数传入了当前的Entry链表），然后直接用这个新的Entry对象取代了旧的Entry链表，看一下Entry的构造方法可以知道是头插法。

```java
Entry(int h, K k, V v, Entry<K,V> n) {
	value = v; next = n; key = k; hash = h;
}
```

从构造方法中的next=n可以看出确实是把原本的链表直接链在了新建的Entry对象的后边，可以断定是插入头部。

### JDK8

还是继续查看put方法的源码查看插入节点的代码：

```java
for (int binCount = 0; ; ++binCount) {
  // e是p的下一个节点
  if ((e = p.next) == null) {
    // 插入链表的尾部
    p.next = newNode(hash, key, value, null);
    // 如果插入后链表长度大于8则转换为红黑树 TREEIFY_THRESHOLD=8
    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
      treeifyBin(tab, hash);
    break;
  }
  if (e.hash == hash &&
      ((k = e.key) == key || (key != null && key.equals(k))))
    break;
  p = e;
}
```

从这段代码中可以很显然地看出当到达链表尾部（即p是链表的最后一个节点）时，e被赋为null，会进入这个分支代码，然后会用newNode方法建立一个新的节点插入尾部。

# 四、HashMap的默认参数理解

**1.为什么HashMap的Entry数组长度默认为16呢?为什么数组长度一定要是2的n次幂呢？**

查看HashMap计算hashcode的方法获取存储的位置：
为了减少hash值的碰撞,需要实现一个尽量均匀分布的hash函数,在HashMap中通过利用key的hashcode值,来进行位运算

![太棒了！HashMap和 ConcurrentHashMap的问题终于总结清楚了](../../images/d0b0a7a0677345269eddfec52ec437ea.png)

存储的流程

```java
/**
 * Returns index for hash code h.
 */
static int indexFor(int h, int length) {
	return h & (length - 1);
}
```

 

**举个例子**
1.计算"book"的hashcode
十进制 : 3029737
二进制 : 101110001110101110 1001

2.HashMap长度是默认的16，length - 1的结果
十进制 : 15
二进制 : 1111

3.把以上两个结果做与运算
101110001110101110 1001 & 1111 = 1001
1001的十进制 : 9,所以 index=9

**结论：hash算法最终得到的index结果,取决于hashcode值的最后几位**
现在,我们假设HashMap的长度是10,重复刚才的运算步骤:

hashcode : 101110001110101110 1001
length - 1 : 1001
index : 1001

------

再换一个hashcode 101110001110101110 1111 试试:
hashcode : 101110001110101110 1111
length - 1 : 1001
index : 1001
从结果可以看出,虽然hashcode变化了,但是运算的结果都是1001,也就是说,当HashMap长度为10的时候,有些index结果的出现几率会更大而有些index结果永远不会出现(比如0111),这样就**不符合hash均匀分布的原则**。

反观长度16或者其他2的幂,length - 1的值是所有二进制位全为1,这种情况下,index的结果等同于hashcode后几位的值，只要输入的hashcode本身分布均匀,hash算法的结果就是均匀的。

**结论：**HashMap的默认长度为16和规定数组长度为2的幂,是为了降低hash碰撞的几率。HashMap 容量为什么总是为 2 的次幂？推荐看下。

**2.HashMap扩容限制的负载因子为什么是0.75呢？为什么不能是0.1或者1呢？**

由HashMap的put方法中实现中的addEntry的实现代码可知当数组长度达到限制条件的阈值就要进行数组的扩容。

**扩容的方式是：**
新建一个长度为之前数组2倍的新的数组，然后将当前的Entry数组中的元素全部传输过去，扩容后的新数组长度为之前的2倍，所以扩容相对来说是个耗资源的操作。

**扩容的触发条件：**
阈值 = 数组默认的长度 x 负载因子（阈值=16x0.75=12）

```java
threshold = (int)(capacity * loadFactor);
void addEntry(int hash, K key, V value, int bucketIndex) {
  if ((size >= threshold) && (null != table[bucketIndex])) {
    resize(2 * table.length);//当size超过临界阈值threshold，并且即将发生哈希冲突时进行扩容
    hash = (null != key) ? hash(key) : 0;
    bucketIndex = indexFor(hash, table.length);
  }

  createEntry(hash, key, value, bucketIndex);
}
```

由上面的内容可知，

- 如果负载因子为0.5甚至更低的可能的话，最后得到的临时阈值明显会很小，这样的情况就会造成分配的内存的浪费，存在多余的没用的内存空间，也不满足了哈希表均匀分布的情况。
- 如果负载因子达到了1的情况，也就是Entry数组存满了才发生扩容，这样会出现大量的哈希冲突的情况，出现链表过长，因此造成get查询数据的效率。
- 因此选择了0.5~1的折中数也就是0.75，均衡解决了上面出现的情况。

# 五、JDK8下的HashMap的实现

区别：

- 使用一个Node数组取代了JDK7的Entry数组来存储数据，这个Node可能是链表结构，也可能是红黑树结构；
- 如果插入的元素key的hashcode值相同，那么这些key也会被定位到Node数组的同一个格子里，如果不超过**8个**使用链表存储；
- 超过8个，会调用treeifyBin函数，将链表转换为红黑树。那么即使所有key的hashcode完全相同，由于红黑树的特点，查找某个特定元素，也只需要O（logn）的开销。 
  JDK8的HashMap

上图是示意图，主要是描述结构，不会达到这个状态的，因为这么多数据的时候早就扩容了。

### put

- 和 Java7 稍微有点不一样的地方就是，Java7 是先扩容后插入新值的，Java8 先插值再扩容，不过这个不重要。

### get

- 计算 key 的 hash 值，根据 hash 值找到对应数组下标: hash & (length-1)
- 判断数组该位置处的元素是否刚好就是我们要找的，如果不是，走第三步
- 判断该元素类型是否是 TreeNode，如果是，用红黑树的方法取数据，如果不是，走第四步 遍历链表，直到找到相等(==或equals)的 key

# 六、CurrentHashMap的原理

> 由于HashMap是线程不同步的，虽然处理数据的效率高，但是在多线程的情况下存在着安全问题，因此设计了CurrentHashMap来解决多线程安全问题。

HashMap在put的时候，插入的元素超过了容量（由负载因子决定）的范围就会触发扩容操作，就是rehash，这个会重新将原数组的内容重新hash到新的扩容数组中，在多线程的环境下，存在同时其他的元素也在进行put操作，如果hash值相同，可能出现同时在同一数组下用链表表示，造成闭环，导致在get时会出现死循环，所以HashMap是线程不安全的。

**JDK7下的CurrentHashMap**

在JDK1.7版本中，ConcurrentHashMap的数据结构是由一个Segment数组和多个HashEntry组成，主要实现原理是实现了**锁分离**的思路解决了多线程的安全问题，如下图所示：

![太棒了！HashMap和 ConcurrentHashMap的问题终于总结清楚了](../../images/7a17c29443d14e79addc9c7e6cfb50fb.png)

CurrentHashMap的结构

Segment数组的意义就是将一个大的table分割成多个小的table来进行加锁，也就是上面的提到的锁分离技术，而每一个Segment元素存储的是HashEntry数组+链表，这个和HashMap的数据存储结构一样。

> ConcurrentHashMap 与HashMap和Hashtable 最大的不同在于：put和 get 两次Hash到达指定的HashEntry，第一次hash到达Segment,第二次到达Segment里面的Entry,然后在遍历entry链表.

**初始化**

ConcurrentHashMap的初始化是会通过位与运算来初始化Segment的大小，用ssize来表示，源码如下所示

```java
private static final int DEFAULT_CONCURRENCY_LEVEL = 16;
private void writeObject(java.io.ObjectOutputStream s) throws java.io.IOException {
    // For serialization compatibility
    // Emulate segment calculation from previous version of this class
    int sshift = 0;
    int ssize = 1;
    while (ssize < DEFAULT_CONCURRENCY_LEVEL) {
      ++sshift;
      ssize <<= 1;
    }
    int segmentShift = 32 - sshift;
    int segmentMask = ssize - 1;
```

因为ssize用位于运算来计算（ssize <<=1），所以Segment的大小取值都是以2的N次方，无关concurrencyLevel的取值，当然concurrencyLevel最大只能用16位的二进制来表示，即65536，换句话说，Segment的大小最多65536个，没有指定concurrencyLevel元素初始化，Segment的大小ssize默认为 DEFAULT_CONCURRENCY_LEVEL =16

每一个Segment元素下的HashEntry的初始化也是按照位与运算来计算，用cap来表示，如下：

```java
int cap = 1;
while (cap < c)
    cap <<= 1
```

上所示，HashEntry大小的计算也是2的N次方（cap <<=1）， cap的初始值为1，所以**HashEntry最小的容量为2**

**put操作**

```java
 static class Segment<K,V> extends ReentrantLock implements Serializable {
   	private static final long serialVersionUID = 2249069246763182397L;
   	final float loadFactor;
   	Segment(float lf) { this.loadFactor = lf; }
 }
```

从上Segment的继承体系可以看出，Segment实现了ReentrantLock,也就带有锁的功能，当执行put操作时，会进行第一次key的hash来定位Segment的位置，如果该Segment还没有初始化，即通过CAS操作进行赋值，然后进行第二次hash操作，找到相应的HashEntry的位置，这里会利用继承过来的锁的特性，在将数据插入指定的HashEntry位置时（链表的尾端），会通过继承ReentrantLock的tryLock（）方法尝试去获取锁，如果获取成功就直接插入相应的位置，如果已经有线程获取该Segment的锁，那当前线程会以自旋的方式去继续的调用tryLock（）方法去获取锁，超过指定次数就挂起，等待唤醒.

### get

ConcurrentHashMap的get操作跟HashMap类似，只是ConcurrentHashMap第一次需要经过一次hash定位到Segment的位置，然后再hash定位到指定的HashEntry，遍历该HashEntry下的链表进行对比，成功就返回，不成功就返回null

### size 返回ConcurrentHashMap元素大小

计算ConcurrentHashMap的元素大小是一个有趣的问题，因为他是并发操作的，就是在你计算size的时候，他还在并发的插入数据，可能会导致你计算出来的size和你实际的size有相差（在你return size的时候，插入了多个数据），要解决这个问题，JDK1.7版本用两种方案

![太棒了！HashMap和 ConcurrentHashMap的问题终于总结清楚了](../../images/15dbb440b251416fb2ba37c8159d2c57.png)



> 1、第一种方案他会使用不加锁的模式去尝试多次计算ConcurrentHashMap的size，最多三次，比较前后两次计算的结果，结果一致就认为当前没有元素加入，计算的结果是准确的.
> 2、第二种方案是如果第一种方案不符合，他就会给每个Segment加上锁，然后计算ConcurrentHashMap的size返回.

**JDK8的ConcurrentHashMap**

JDK1.8的实现已经摒弃了Segment的概念，而是直接用Node数组+链表+红黑树的数据结构来实现，并发控制使用Synchronized和CAS来操作，整个看起来就像是优化过且线程安全的HashMap，虽然在JDK1.8中还能看到Segment的数据结构，但是已经简化了属性，只是为了兼容旧版本.

![太棒了！HashMap和 ConcurrentHashMap的问题终于总结清楚了](../../images/57d95a15a5d34c91a8cd56fc662e22dd.png)

JDK8的ConcurrentHashMap

在深入JDK1.8的put和get实现之前要知道一些常量设计和数据结构，这些是构成ConcurrentHashMap实现结构的基础，下面看一下基本属性：

```java
// node数组最大容量：2^30=1073741824
private static final int MAXIMUM_CAPACITY = 1 << 30;
// 默认初始值，必须是2的幕数
private static final int DEFAULT_CAPACITY = 16
//数组可能最大值，需要与toArray（）相关方法关联
static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
//并发级别，遗留下来的，为兼容以前的版本
private static final int DEFAULT_CONCURRENCY_LEVEL = 16;
// 负载因子
private static final float LOAD_FACTOR = 0.75f;
// 链表转红黑树阀值,> 8 链表转换为红黑树
static final int TREEIFY_THRESHOLD = 8;
//树转链表阀值，小于等于6（tranfer时，lc、hc=0两个计数器分别++记录原bin、新binTreeNode数量，<=UNTREEIFY_THRESHOLD 则untreeify(lo)）
static final int UNTREEIFY_THRESHOLD = 6;
static final int MIN_TREEIFY_CAPACITY = 64;
private static final int MIN_TRANSFER_STRIDE = 16;
private static int RESIZE_STAMP_BITS = 16;
// 2^15-1，help resize的最大线程数
private static final int MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1;
// 32-16=16，sizeCtl中记录size大小的偏移量
private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;
// forwarding nodes的hash值
static final int MOVED     = -1;
// 树根节点的hash值
static final int TREEBIN   = -2;
// ReservationNode的hash值
static final int RESERVED  = -3;
// 可用处理器数量
static final int NCPU = Runtime.getRuntime().availableProcessors();
//存放node的数组
transient volatile Node<K,V>[] table;
/*控制标识符，用来控制table的初始化和扩容的操作，不同的值有不同的含义
 *当为负数时：-1代表正在初始化，-N代表有N-1个线程正在 进行扩容
 *当为0时：代表当时的table还没有被初始化
 *当为正数时：表示初始化或者下一次进行扩容的大小
private transient volatile int sizeCtl;
```

**JDK8的Node**

Node是ConcurrentHashMap存储结构的基本单元，继承于HashMap中的Entry，用于存储数据，**Node数据结构很简单，就是一个链表，但是只允许对数据进行查找，不允许进行修改**源代码如下：

```java
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        volatile V val;
        volatile Node<K,V> next;

        Node(int hash, K key, V val, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.val = val;
            this.next = next;
        }

        public final K getKey()       { return key; }
        public final V getValue()     { return val; }
        public final int hashCode()   { return key.hashCode() ^ val.hashCode(); }
        public final String toString(){ return key + "=" + val; }
        public final V setValue(V value) {
            throw new UnsupportedOperationException();
        }

        public final boolean equals(Object o) {
            Object k, v, u; Map.Entry<?,?> e;
            return ((o instanceof Map.Entry) &&
                    (k = (e = (Map.Entry<?,?>)o).getKey()) != null &&
                    (v = e.getValue()) != null &&
                    (k == key || k.equals(key)) &&
                    (v == (u = val) || v.equals(u)));
        }

        /**
         * Virtualized support for map.get(); overridden in subclasses.
         */
        Node<K,V> find(int h, Object k) {
            Node<K,V> e = this;
            if (k != null) {
                do {
                    K ek;
                    if (e.hash == h &&
                        ((ek = e.key) == k || (ek != null && k.equals(ek))))
                        return e;
                } while ((e = e.next) != null);
            }
            return null;
        }
    }
```

**TreeNode**

TreeNode继承于Node，但是数据结构换成了二叉树结构，它是红黑树的数据的存储结构，用于红黑树中存储数据，**当链表的节点数大于8**时会转换成红黑树的结构，他就是通过TreeNode作为存储结构代替Node来转换成黑红树源代码如下

```java
static final class TreeNode<K,V> extends Node<K,V> {
        TreeNode<K,V> parent;  // red-black tree links
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;

        TreeNode(int hash, K key, V val, Node<K,V> next,
                 TreeNode<K,V> parent) {
            super(hash, key, val, next);
            this.parent = parent;
        }

        Node<K,V> find(int h, Object k) {
            return findTreeNode(h, k, null);
        }

        /**
         * Returns the TreeNode (or null if not found) for the given key
         * starting at given root.
         */
        final TreeNode<K,V> findTreeNode(int h, Object k, Class<?> kc) {
            if (k != null) {
                TreeNode<K,V> p = this;
                do  {
                    int ph, dir; K pk; TreeNode<K,V> q;
                    TreeNode<K,V> pl = p.left, pr = p.right;
                    if ((ph = p.hash) > h)
                        p = pl;
                    else if (ph < h)
                        p = pr;
                    else if ((pk = p.key) == k || (pk != null && k.equals(pk)))
                        return p;
                    else if (pl == null)
                        p = pr;
                    else if (pr == null)
                        p = pl;
                    else if ((kc != null ||
                              (kc = comparableClassFor(k)) != null) &&
                             (dir = compareComparables(kc, k, pk)) != 0)
                        p = (dir < 0) ? pl : pr;
                    else if ((q = pr.findTreeNode(h, k, kc)) != null)
                        return q;
                    else
                        p = pl;
                } while (p != null);
            }
            return null;
        }
    }
```

### TreeBin

TreeBin从字面含义中可以理解为存储树形结构的容器，而树形结构就是指TreeNode，所以TreeBin就是封装TreeNode的容器，它提供转换黑红树的一些条件和锁的控制.

# 总结和思考

其实可以看出JDK1.8版本的ConcurrentHashMap的数据结构已经接近HashMap，相对而言，ConcurrentHashMap只是增加了同步的操作来控制并发，从JDK1.7版本的ReentrantLock+Segment+HashEntry，到JDK1.8版本中synchronized+CAS+HashEntry+红黑树,相对而言，总结如下思考

- JDK1.8的实现降低锁的粒度，JDK1.7版本锁的粒度是基于Segment的，包含多个HashEntry，而JDK1.8锁的粒度就是HashEntry（首节点）
- JDK1.8版本的数据结构变得更加简单，使得操作也更加清晰流畅，因为已经使用synchronized来进行同步，所以不需要分段锁的概念，也就不需要Segment这种数据结构了，由于粒度的降低，实现的复杂度也增加了
- JDK1.8使用红黑树来优化链表，基于长度很长的链表的遍历是一个很漫长的过程，而红黑树的遍历效率是很快的，代替一定阈值的链表，这样形成一个最佳拍档
- JDK1.8为什么使用内置锁synchronized来代替重入锁ReentrantLock，我觉得有以下几点
  1.因为粒度降低了，在相对而言的低粒度加锁方式，synchronized并不比ReentrantLock差，在粗粒度加锁中ReentrantLock可能通过Condition来控制各个低粒度的边界，更加的灵活，而在低粒度中，Condition的优势就没有了2.JVM的开发团队从来都没有放弃synchronized，而且基于JVM的synchronized优化空间更大，使用内嵌的关键字比使用API更加自然3.在大量的数据操作下，对于JVM的内存压力，基于API的ReentrantLock会开销更多的内存，虽然不是瓶颈，但是也是一个选择依据