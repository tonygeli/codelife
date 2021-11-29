[TOC]



1.为什么spring循环依赖需要3级缓存

2.本地方法栈如何理解







▲ 38 HashMap 与 ConcurrentHashMap 的实现原理是怎样的？ConcurrentHashMap 是如何保证线程安全的？

▲ 29 synchronized 关键字底层是如何实现的？它与 Lock 相比优缺点分别是什么？

▲ 24 简述 JVM 的内存模型 JVM 内存是如何对应到操作系统内存的？

▲ 20 集合类中的 List 和 Map 的线程安全版本是什么，如何保证线程安全的？

▲ 14 Java 线程和操作系统的线程是怎么对应的？Java线程是怎样进行调度的?

▲ 14 ThreadLocal 实现原理是什么？

▲ 11 简述 BIO, NIO, AIO 的区别

▲ 8 简述 Spring AOP 的原理

▲ 6 简述 Synchronized，Volatile，可重入锁的不同使用场景及优缺点

▲ 2 简述 Java 的 happen before 原则

▲ 1 SpringBoot 是如何进行自动配置的？







字节码编译过程？

2.0-spring的上下文切换，如何优化上下文来减少系统资源的消耗？
springboot自动配置的原理？
spring的自动装配？
Springmvc的工作原理？
spring如何解决循环依赖？
spring事务传播行为？



4.0-redis如何实现延时队列？
redis崩溃时如何处理数据？
5.0-hashmap数据结构？



JAVA基础  https://blog.csdn.net/guorui_java/article/details/108153368

https://www.zhihu.com/question/60949531





HashMap底层实现原理，什么时候变成红黑树

jvm如何排查线上问题 内存溢出 线程池超出问题

线程池原理 经常用那些线程池 为什么

并发场景 重入锁 同步锁



redis用过那些数据结构 用redis队列来做延迟队列有什么优劣势









kafka如何保证消息不丢失 不重复消费

kafka消息积压怎么处理 怎么产生的消息积压？



## 0快问快答

### private修饰的⽅法可以通过反射访问，那么private的意义是什么 

​	1.对用户常规使用java的约束 2.从外部对对象调用，看清类结构

### Java类初始化顺序 

​	基类静态代码块，基类静态成员字段——>派生类静态代码块，派生类静态成员字段——>基类普通代码块，基类普通成员字段——>基类构造函数——>派生类普通代码块，派生类普通成员字段——>派生类构造函数

### 对⽅法区和永久区的理解以及它们之间的关系

​	方法区是jvm规范里要求的，永久区是Hotspot虚拟机对方法区的具体实现，前者是规范，后者是实现方式。

```xml
-XX:NewSize=1024m：设置年轻代初始值为1024M。
-XX:PermSize=256m：设置持久代初始值为256M。
-XX:MaxPermSize=256m：设置持久代最大值为256M。
-XX:SurvivorRatio=4：设置年轻代中Eden区与Survivor区的比值。表示2个Survivor区（JVM堆内存年轻代中默认有2个大小相等的Survivor区）与1个Eden区的比值为2:4，即1个Survivor区占整个年轻代大小的1/6。

JDK8中用metaspace代替permsize，因此在许多我们设置permsize大小的地方同样需要修改配置为metaspace
将-XX:PermSize=200m;-XX:MaxPermSize=256m;
=======> 修改为：-XX:MetaspaceSize=200m;-XX:MaxMetaspaceSize=256m;
```





### ⼀个java⽂件有3个类，编译后有⼏个class⽂件

​	几个类编译后就有几个class类

### 局部变量使⽤前需要显式地赋值，否则编译通过不了，为什么这么设计

​	**这样设计是一种约束**，尽最大程度减少使用者犯错的可能。

### ReadWriteLock读写之间互斥吗 

​	除了读和读之间是共享的，其它都是互斥的

### Semaphore拿到执⾏权的线程之间是否互斥

​	Semaphore拿到执⾏权的线程之间是否互斥，Semaphore、CountDownLatch、CyclicBarrier、 Exchanger 为java并发编程的4个辅助类，⾯试中常问的 CountDownLatch CyclicBarrier之间的区别，⾯试 者肯定是经常碰到的， 所以问起来意义不⼤，Semaphore问的相对少⼀些，有些知识点如果没有使⽤过还 是会忽略，Semaphore可有多把锁，可允许多个线程同时拥有执⾏权，这些有执⾏权的线程如并发访问同⼀对象，会产⽣线程安全问题。



### 写⼀个你认为最好的单例模式 

​	双检锁的方式	

```java
public class Singleton {
	private Singleton() {}
	private volatile static Singleton instance;
	public static Singleton getInstance() {
		if (null == instance) {
			synchronized (Singleton.class) {
				if (null == instance) {
					instance = new Singleton();
				}
			}
		}
		return instance;
	}}
```

### B树和B+树是解决什么样的问题的，怎样演化过来，之间区别

​	B树和B+树，这题既问mysql索引的实现原理，也问数据结构基础，首先从二叉树说起，因为会产生退化现象，提出了平衡二叉树，再提出怎样让每一层放的节点多一些来减少遍历高度，引申出m叉树，m叉搜索树同样会有退化现象，引出m叉平衡树，也就是B树，这时候每个节点既放了key也放了value，怎样使每个节点放尽可能多的key值，以减少遍历高度呢（访问磁盘次数），可以将每个节点只放key值，将value值放在叶子结点，在叶子结点的value值增加指向相邻节点指针，这就是优化后的B+树。然后谈谈数据库索引失效的情况，为什么给离散度低的字段（如性别）建立索引是不可取的，查询数据反而更慢，如果将离散度高的字段和性别建立联合索引会怎样，有什么需要注意的？

### 写⼀个⽣产者消费者模式

​	生产者消费者模式，synchronized锁住一个LinkedList，一个生产者，只要队列不满，生产后往里放，一个消费者只要队列不空，向外取，两者通过wait()和notify()进行协调，写好了会问怎样提高效率，最后会聊一聊消息队列设计精要思想及其使用。

synchronized代码块通过javap生成的字节码中包含 **monitorenter** 和 **monitorexit**指令。

```java
public static void main(String[] args) {
        final Object lock = new Object();
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("1.thread A is waiting to get lock");
                synchronized (lock) {
                		try{		// "2 thread A get lock"
                        TimeUnit.SECONDS.sleep(1);	// "3 thread A do wait method"
                        lock.wait();
                        System.out.println("7 wait end");
                    }catch(InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }).start();
        new Thread(new Runnable() {
            @Override
            public void run () {
                System.out.println("4 thread B is waiting to get lock");
                synchronized (lock) {	// "5 thread B get lock"
                    TimeUnit.SECONDS.sleep(5);
                    lock.notify();	// "6 thread B do notify method"
                }
            }
        }).start();
    }

```

在多核环境下，线程A和B有可能同时执行monitorenter指令，并获取lock对象关联的monitor，只有一个线程可以和monitor建立关联，假设线程A执行加锁成功；
notifyAll方法的注释：notifyAll方法会唤醒monitor的wait set中所有线程。
**什么是monitor？**
在HotSpot虚拟机中，monitor采用ObjectMonitor实现。

```java
ObjectMonitor {
	_header = null;
  _count = 0;
  _waiters = 0;
  _owner = null;		// 
  _WaitSet = null;	// 处于wait状态的线程，会被加入到wait set；
  _EntryList = null;	// 处于等待锁block状态的线程，会被加入到entry set；
  OwnerIsThread = 0;
}
```

**wait方法实现**
1、将当前线程封装成ObjectWaiter对象node；
2、通过ObjectMonitor::AddWaiter方法将node添加到_WaitSet列表中
3、通过ObjectMonitor::exit方法释放当前的ObjectMonitor对象，这样其它竞争线程就可以获取该ObjectMonitor对象。
4、最终底层的park方法会挂起线程；

**notify方法实现**
1、如果当前_WaitSet为空，即没有正在等待的线程，则直接返回；
2、通过ObjectMonitor::DequeueWaiter方法，获取_WaitSet列表中的第一个ObjectWaiter节点，实现也很简单。
这里需要注意的是，在jdk的notify方法注释是随机唤醒一个线程，其实是WaitSet列表第一个ObjectWaiter
3、根据不同的策略，将取出来的ObjectWaiter节点，加入到_EntryList或则通过

**notifyAll方法实现**
lock.notifyAll()方法最终通过ObjectMonitor的void notifyAll(TRAPS)实现：
通过for循环取出_WaitSet的ObjectWaiter节点，并根据不同策略，加入到_EntryList或则进行自旋操作。
从JVM的方法实现中，可以发现：notify和notifyAll并不会释放所占有的ObjectMonitor对象，其实真正释放ObjectMonitor对象的时间点是在执行monitorexit指令，一旦释放ObjectMonitor对象了，entry set中ObjectWaiter节点所保存的线程就可以开始竞争ObjectMonitor对象进行加锁操作了ßßß



### 写⼀个死锁 

````java
public class DeadLock {
	public static void main() {
    new Thread(new Runnable() {
      @Override
      public void run() {
        synchronized(lock1) {
          synchronized(lock2) {}
        }
      }
    }).start();
    
    new Thread(new Runnable(){
      @Override
      public void run() {
        synchronized(lock2) {
          synchronized(lock1) {}
        }
      }
    }).start();
  }
}
````

### cpu 100%怎样定位 

```shell
// 先用top定位最耗cpu的java进程 例如: 12430
$ top或者 htop（高级）   键入 P （大写P），按照cpu进行排序
$ top -p 12430 -H 定位到最耗cpu的线程 的ID 例如：12483
$ top -Hp 1865 ，显示一个进程的线程运行信息列表  键入P (大写p)，线程按照CPU使用率排序
把第二步定位的线程ID，转成16进制，printf "%x" 12483 得到 ：30c3
从jstack 输出的线程快照中找到线程的对堆栈信息 jstack 12430 |grep 30c3 -A 60 |less
方法：jstack 10765 | grep ‘0x2a34’ -C5 --color`
```



### String a = "ab"; String b = "a" + "b"; a == b 是否相等，为什么 

​	常量池	

### 可以⽤for循环直接删除ArrayList的特定元素吗？可能会出现什么问题？怎样解决

​	解决方案：用 Iterator。ArrayList中的remove方法会执行System.arraycopy方法，导致删除元素时涉及到数组元素的移动

### 新的任务提交到线程池，线程池是怎样处理 

​	第一步 ：线程池判断核心线程池里的线程是否都在执行任务。如果不是，则创建一个新的工作线程来执行任务。如果核心线程池里的线程都在执行任务，则执行第二步。
​	第二步 ：线程池判断工作队列是否已经满。如果工作队列没有满，则将新提交的任务存储在这个工作队列里进行等待。如果工作队列满了，则执行第三步。
 	第三步 ：线程池判断线程池的线程是否都处于工作状态。如果没有，则创建一个新的工作线程来执行任务。如果已经满了，则交给饱和策略来处理这个任务。

### AQS和CAS原理 

​	抽象队列同步器AQS（AbstractQueuedSychronizer），如果说java.util.concurrent的基础是CAS的话，那么AQS就是整个Java并发包的核心了，ReentrantLock、CountDownLatch、Semaphore等都用到了它。AQS实际上以双向队列的形式连接所有的Entry，比方说ReentrantLock，所有等待的线程都被放在一个Entry中并连成双向队列，前面一个线程使用ReentrantLock好了，则双向队列实际上的第一个Entry开始运行。AQS定义了对双向队列所有的操作，而只开放了tryLock和tryRelease方法给开发者使用，开发者可以根据自己的实现重写tryLock和tryRelease方法，以实现自己的并发功能。
　比较并替换CAS(Compare and Swap)，假设有三个操作数：内存值V、旧的预期值A、要修改的值B，当且仅当预期值A和内存值V相同时，才会将内存值修改为B并返回true，否则什么都不做并返回false，整个比较并替换的操作是一个原子操作。CAS一定要volatile变量配合，这样才能保证每次拿到的变量是主内存中最新的相应值，否则旧的预期值A对某条线程来说，永远是一个不会变的值A，只要某次CAS操作失败，下面永远都不可能成功。
​	CAS虽然比较高效的解决了原子操作问题，但仍存在三大问题：循环时间长开销很大。只能保证一个共享变量的原子操作。ABA问题。

### synchronized底层实现原理 

​	synchronized (this)原理：涉及两条指令：monitorenter，monitorexit；再说同步方法，从同步方法反编译的结果来看，方法的同步并没有通过指令monitorenter和monitorexit来实现，相对于普通方法，其常量池中多了ACC_SYNCHRONIZED标示符。
　JVM就是根据该标示符来实现方法的同步的：当方法被调用时，调用指令将会检查方法的 ACC_SYNCHRONIZED 访问标志是否被设置，如果设置了，执行线程将先获取monitor，获取成功之后才能执行方法体，方法执行完后再释放monitor。在方法执行期间，其他任何线程都无法再获得同一个monitor对象。
　这个问题会接着追问：java对象头信息，偏向锁，轻量锁，重量级锁及其他们相互间转化。

### volatile作⽤，指令重排相关

​	理解volatile关键字的作用的前提是要理解Java内存模型，volatile关键字的作用主要有两点：
​	1，多线程主要围绕可见性和原子性两个特性而展开，使用volatile关键字修饰的变量，保证了其在**多线程之间的可见性**，即每次读取到volatile变量，一定是最新的数据
​	2，代码底层执行不像我们看到的高级语言—-Java程序这么简单，它的执行是Java代码–>字节码–>根据字节码执行对应的C/C++代码–>C/C++代码被编译成汇编语言–>和硬件电路交互，现实中，为了获取更好的性能JVM可能会对指令进行重排序，多线程下可能会出现一些意想不到的问题。使用volatile则会对**禁止语义重排序**，当然这也一定程度上降低了代码执行效率
​	从实践角度而言，volatile的一个重要作用就是和CAS结合，保证了原子性，详细的可以参见java.util.concurrent.atomic包下的类，比如AtomicInteger。



### AOP和IOC原理 

​	AOP 和 IOC是Spring精华部分，AOP可以看做是对OOP的补充，对代码进行横向的扩展，通过代理模式实现，代理模式有静态代理，动态代理，Spring利用的是动态代理，在程序运行过程中将增强代码织入原代码中。IOC是控制反转，将对象的控制权交给Spring框架，用户需要使用对象无需创建，直接使用即可。AOP和IOC最可贵的是它们的思想。

### Spring怎样解决循环依赖的问题

​	什么是循环依赖，怎样检测出循环依赖，Spring循环依赖有几种方式，使用基于setter属性的循环依赖为什么不会出现问题，接下来会问：Bean的生命周期。

### dispatchServlet怎样分发任务的 

![img](../../images/955092-20201110163718380-2119784231-7071543.jpg)

1）. 用户发请求-->DispatcherServlet，前端控制器收到请求后自己不进行处理，而是委托给其他的解析器进行处理，作为统一访问点，进行全局的流程控制。

2）.DispatcherServlet-->HandlerMapping，HandlerMapping将会把请求映射为HandlerExecutionChain对象（包含一个Handler处理器,多个HandlerInterceptor拦截器)。

3）.DispatcherServlet-->HandlerAdapter,HandlerAdapter将会把处理器包装为适配器，从而支持多种类型的处理器。

4）.HandlerAdapter-->处理器功能处理方法的调用，HandlerAdapter将会根据适配的结果调用真正的处理器的功能处理方法，完成功能处理，并返回一个ModelAndView对象(包含模型数据，逻辑视图名)

5）.ModelAndView的逻辑视图名-->ViewResolver，ViewResoler将把逻辑视图名解析为具体的View。

6）.View-->渲染，View会根据传进来的Model模型数据进行渲染，此处的Model实际是一个Map数据结构

7）.返回控制权给DispatcherServlet，由DispatcherServlet返回响应给用户。

### mysql给离散度低的字段建⽴索引会出现什么问题，具体说下原因

 先上结论：重复性较强的字段，不适合添加索引。mysql给离散度低的字段，比如性别设置索引，再以性别作为条件进行查询反而会更慢。

　　一个表可能会涉及两个数据结构(文件)，一个是表本身，存放表中的数据，另一个是索引。索引是什么？它就是把一个或几个字段（组合索引）按规律排列起来，再附上该字段所在行数据的物理地址（位于表中）。比如我们有个字段是年龄，如果要选取某个年龄段的所有行，那么一般情况下可能需要进行一次全表扫描。但如果以这个年龄段建个索引，那么索引中会按年龄值根据特定数据结构建一个排列，这样在索引中就能迅速定位，不需要进行全表扫描。为什么性别不适合建索引呢？因为访问索引需要付出额外的IO开销，从索引中拿到的只是地址，要想真正访问到数据还是要对表进行一次IO。假如你要从表的100万行数据中取几个数据，那么利用索引迅速定位，访问索引的这IO开销就非常值了。但如果是从100万行数据中取50万行数据，就比如性别字段，那你相对需要访问50万次索引，再访问50万次表，加起来的开销并不会比直接对表进行一次完整扫描小。

　　当然如果把性别字段设为表的聚集索引，那么就肯定能加快大约一半该字段的查询速度了。聚集索引指的是表本身数据按哪个字段的值来进行排序。因此，聚集索引只能有一个，而且使用聚集索引不会付出额外IO开销。当然你得能舍得把聚集索引这么宝贵资源用到性别字段上。

　　可以根据业务场景需要，将性别和其它字段建立联合索引，比如时间戳，但是建立索引记得把时间戳字段放在性别前面。

### 怎么做到的热加载？

热部署和热加载都是基于类加载器实现的，热加载是服务器监听class等文件的改变然后对改变的文件进行局部加载，所以不会删除session，也不会释放内存。热部署就是全局部署，会清空session以及释放内存。自定义加载器实现热加载

用户自定义加载器需要继承ClassLoader，实现原理就是通过一个线程去监听文件的修改时间，然后重写findClass方法，把文件以流的形式读进来，然后调defineClass方法。在JDK1.2之后，双亲委派模式已经被引入到类加载体系中，因此不建议重写loadClass方法，只需要重写findClass就可以了

```java
private bype[] loadClassData(String name) {
  FileInputStream inputStream = new FileInputStream(new File());
  ByteOutputStream outStream = new ByteOutputStream();
  int b = 0;
  while ((b = inputStream.read()) != -1) {
    outStream.write(b);
  }
  inputStream.close();
  return outStream.toByteArray();
}
```



### 分布式唯一ID是怎么实现的





### HashMap 与 ConcurrentHashMap 的实现原理是怎样的？ConcurrentHashMap 是如何保证线程安全的？

jdk1.7中是采用Segment + HashEntry + ReentrantLock的方式进行实现的，而1.8中放弃了Segment臃肿的设计，取而代之的是采用Node + CAS + Synchronized来保证并发安全进行实现

一、使用volatile**保证**当Node中的值变化时对于其他**线程是**可见的

二、使用table数组的头结点作为synchronized的锁来**保证**写操作的**安全**

三、当头结点为null时，使用CAS操作来**保证**数据能正确的写入。

ConcurrentHashMap中维护着一个Segment数组，每个Segment可以看做是一个HashMap。Segment本身继承了ReentrantLock，它本身就是一个锁。

```java
private static final int DEFAULT_CONCURRENCY_LEVEL = 16;
while (ssize < DEFAULT_CONCURRENCY_LEVEL) {
  ssize <<= 1;
}
static class Segment<K,V> extends ReentrantLock {
  
}

```











### Java 中垃圾回收机制中如何判断对象需要回收？常见的 GC 回收算法有哪些？

1.引用计数算法：两个对象互相引用就无法回收
2.可达性分析算法：“GC Roots”的对象作为起始点，没有任何引用链相连则证明对象是不可用的
	虚拟机栈 引用的对象
	方法区中类静态属性 引用的对象
	方法区常量 引用的对象
	本地方法栈中JNI（即一般说的Native方法）引用的对象

•**强引用**，就是指在程序代码之中普遍存在的，类似A a = new A()这样的引用，只要强引用存在，垃圾回收器就不会回收掉被引用的对象；
•**软引用**，用来描述一些还有用但并非必须的对象，对于软引用的对象，在系统将要发生内存溢出异常之前，将会把这些对象列入回收范围之中进行第二次回收。如果这次回收还没有足够的内存，才会出现内存溢出异常；
•**弱引用**，也是用来描述非必需的对象，但是它的强度比软引用更弱，被弱引用关联的对象只能生存到下一次回收发生之前，当垃圾回收器工作时，无论当前内存是否足够，都会回收掉；
•**虚引用**，它是最弱的一种引用关系，一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用取得一个对象实例、为一个对象设置引用关联的唯一目的就是能在这个对象被收集器回收时收到一个系统通知。

#### G1垃圾回收器：

**G1垃圾回收器相关数据结构**

1.在HotSpot的实现中，整个堆被划分成2048左右个Region。每个Region的大小在1-32MB之间，具体多大取决于堆的大小。
2.对于Region来说，它会有一个分代的类型，并且是唯一一个。即，每一个Region，它要么是young的，要么是old的。还有一类十分特殊的Humongous。所谓的Humongous，就是一个对象的大小超过了某一个阈值——HotSpot中是Region的1/2，那么它会被标记为Humongous。

![img](../../images/format,png.png)
每一个分配的Region，都可以分成两个部分，已分配的和未被分配的。它们之间的界限被称为top。总体上来说，把一个对象分配到Region内，只需要简单增加top的值。这个做法实际上就是bump-the-pointer。
即每一次回收都是回收N个Region。这个N是多少，主要受到G1回收的效率和用户设置的软实时目标有关。每一次的回收，G1会选择可能回收最多垃圾的Region进行回收。与此同时，G1回收器会维护一个空间Region的链表。每次回收之后的Region都会被加入到这个链表中。
每一次都只有一个Region处于被分配的状态中，被称为current region。在多线程的情况下，这会带来并发的问题。G1回收器采用和CMS一样的TLABs的手段。即为每一个线程分配一个Buffer，线程分配内存就在这个Buffer内分配。但是当线程耗尽了自己的Buffer之后，需要申请新的Buffer。这个时候依然会带来并发的问题。G1回收器采用的是CAS（Compate And Swap）操作。
3.卡片 Card
在每个分区内部又被分成了若干个大小为512 Byte卡片(Card)，标识堆内存最小可用粒度所有分区的卡片将会记录在全局卡片表(Global Card Table)中，分配的对象会占用物理上连续的若干个卡片，当查找对分区内对象的引用时便可通过记录卡片来查找该引用对象(见RSet)。每次对内存的回收，都是对指定分区的卡片进行处理。
4.RS(Remember Set)
RS(Remember Set)是一种抽象概念，用于记录从非收集部分指向收集部分的指针的集合。
在传统的分代垃圾回收算法里面，RS(Remember Set)被用来记录分代之间的指针。在G1回收器里面，RS被用来记录从其他Region指向一个Region的指针情况。因此，一个Region就会有一个RS。这种记录可以带来一个极大的好处：在回收一个Region的时候不需要执行全堆扫描，只需要检查它的RS就可以找到外部引用，而这些引用就是initial mark的根之一。

那么，如果一个线程修改了Region内部的引用，就必须要去通知RS，更改其中的记录。为了达到这种目的，G1回收器引入了一种新的结构，CT(Card Table)——卡表。每一个Region，又被分成了固定大小的若干张卡(Card)。每一张卡，都用一个Byte来记录是否修改过。卡表即这些byte的集合。实际上，如果把RS理解成一个概念模型，那么CT就可以说是RS的一种实现方式。

在RS的修改上也会遇到并发的问题。因为一个Region可能有多个线程在并发修改，因此它们也会并发修改RS。为了避免这样一种冲突，G1垃圾回收器进一步把RS划分成了多个哈希表。每一个线程都在各自的哈希表里面修改。最终，从逻辑上来说，RS就是这些哈希表的集合。哈希表是实现RS的一种通常的方式之一。它有一个极大的好处就是能够去除重复。这意味着，RS的大小将和修改的指针数量相当。而在不去重的情况下，RS的数量和写操作的数量相当。

4.2.2 G1垃圾回收器执行步骤：

1、初始标记；
初始标记阶段仅仅只是标记一下GC Roots能直接关联到的对象，并且修改TAMS的值，让下一个阶段用户程序并发运行时，能在正确可用的Region中创建新对象，这一阶段需要停顿线程，但是耗时很短，

2、并发标记；
并发标记阶段是从GC Root开始对堆中对象进行可达性分析，找出存活的对象，这阶段时耗时较长，但可与用户程序并发执行。

3、最终标记；
最终标记阶段则是为了修正在并发标记期间因用户程序继续运作而导致标记产生变动的那一部分标记记录，虚拟机将这段时间对象变化记录在线程Remenbered Set Logs里面，最终标记阶段需要把Remembered Set Logs的数据合并到Remembered Set Logs里面，最终标记阶段需要把Remembered Set Logs的数据合并到Remembered Set中，这一阶段需要停顿线程，但是可并行执行。

4、筛选回收
最后在筛选回收阶段首先对各个Region的回收价值和成本进行排序，根据用户所期望的GC停顿时间来制定回收计划。




### 1、HashMap了解吗？说说底层的数据结构，以及put方法中详细的流程

jdk1.8后数组+链表+红黑树的结构

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
        Node<K,V>[] tab;  //缓存底层数组用,都是指向一个地址的引用
        Node<K,V> p;      //插入数组的桶i处的键值对节点
        int n;            //底层数组的长度
        int i;            //插入数组的桶的下标
        //刚开始table是null或空的时候，初始化个默认的table；为tab和n赋值，tab指向底层数组，n为底层数组的长度
        if ((tab = table) == null || (n = tab.length) == 0){
            n = (tab = resize()).length;
        }
        //(n - 1) & hash：根据hash值算出插入点在底层数组的桶的位置，即下标值；为p赋值，也为i赋值（不管碰撞与否，都已经赋值了）
        //如果在数组上，没有发生碰撞，即当前要插入的位置上之前没有插入过值，则直接在此位置插入要插入的键值对
        if ((p = tab[i = (n - 1) & hash]) == null){
            tab[i] = newNode(hash, key, value, null);//插入的节点的next属性是null
        } else {    //发生碰撞，即当前位置已经插入了值
            Node<K,V> e;    //中间变量吧，跟冒泡排序里面的那个中间变量似的，起到个值交换的作用
            K k;            //同上
            //hash值相同，key也相同，那么就是更新这个键值对的值。同 jdk 1.7
            if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k)))){ //注意在这个if内【e != null】
                e = p;//这地方，e = p  他们两个都是指向数组下标为i的地方，在这if else if else结束之后，再把节点的值value给更新了
            } else if (p instanceof TreeNode){  //这个树方法可能会返回null。
                //jdk 1.8引入了红黑树来处理碰撞，上面判断p的类型已经是树结构了，
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);//如果是，则走添加树的方法。
            } else {    //注意在这个else内，当为添加新节点时，【e == 】；更新某个节点时，就不是null
                for (int binCount = 0; ; ++binCount) {//还未形成树结构，还是jdk 1.7的链表结构
                    //差别就是1.7:是头插法，后来的留在数组上，先来的链在尾上；1.8:是先来的就留在数组上，后来的链在尾上
                    //判断p.next是否为空，同时为e赋值，若为空，则p.next指向新添加的节点，这是在链表长度小于7的时候
                    if ((e = p.next) == null) {
                        //这个地方有个不好理解的地方：在判断条件里面，把e指向p.next，也就是说现在e=null而不是下下一行错误的理解。
                        //这也就解释了更新的时候，返回oldValue，新建的时候，是不在那地方返回的。
                        p.next = newNode(hash, key, value, null);//e = p.next,p.next指向新生成的节点，也就是说e指向新节点（错误）
                        //对于临界值的分析：
                        //假设此次是第六次，binCount == 6,不会进行树变，当前链表长度是7；下次循环。
                        //binCount == 7，条件成立，进行树变，以后再put到这个桶的位置的时候，这个else就不走了，走中间的那个数结构的分叉语句啦
                        //这个时候，长度为8的链表就变成了红黑树啦
                        if (binCount >= TREEIFY_THRESHOLD - 1){// -1 for 1st //TREEIFY_THRESHOLD == 8
                            treeifyBin(tab, hash);
                        }
                        break;//插入新值或进行树变后，跳出for循环。此时e未重定向，还是指向null，虽然后面p.next指向了新节点。
                        //但是，跟e没关系。
                    }
                    //如果在循环链表的时候，找到key相同的节点，那么就跳出循环，就走不到链表的尾上了。
                    // e已经在上一步已经赋值了，且不为null,也会跳出for循环，会在下面更新value的值
                    if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))){
                        break;
                    }
                    //这个就是p.next也就是e不为空，然后，还没有key相同的情况出现，那就继续循环链表，
                    // p指向p.next也就是e，继续循环，继续，e=p.next
                    p = e;
                    //直到p.next为空，添加新的节点；或者出现key相等，更新旧值的情况才跳出循环。
                }
            }
            //经过上面if else if else之后，e在新建节点的时候，为null；更新的时候，则被赋值。在树里面处理putTreeVal（）同样如此，
            if (e != null) { // existing mapping for key//老外说的对，就是只有更新的时候，才走这，才会直接return oldValue
                V oldValue = e.value;
                //onlyIfAbsent 这个在调用hashMap的put()的时候，一直是false，那么下面更新value是肯定执行的
                if (!onlyIfAbsent || oldValue == null){
                    e.value = value;
                }
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold){
            resize();
        }
        afterNodeInsertion(evict);
        return null;
    }
```

### 2、Spring容器注入bean的方式有哪些

1.@Component注解，加@ComponentScan扫描

2.配置类中加@Bean



Spring 为啥把bean默认设计成单例？
答案：为了提高性能！！！从几个方面：1.少创建实例2.垃圾回收3.缓存快速获取

### 3、说说线程池的参数，工作原理以及拒绝策略

7个参数：核心线程数、最大线程数、活跃时间、时间单位、阻塞队列、线程工厂、拒绝策略
3个方法：newSingleThreadExecutor  newFixedThreadPool newCachedThreadPool
4个策略 AbortPolicy抛弃异常 CallerRunsPolicy   DiscardPolicy    DiscardOldestPolicy

```java
class ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler)
```

### 4、Spring事务级别有哪些？

### 5、说说服务注册中心的原理，比如Eureka（他们公司用的这个）

### Mysql的事务隔离级别知道吗？底层的实现原理是什么？

### 什么是聚簇索引？

### HashMap的长度为什么要是2的N次方？

h%len    h&(len-1)

### Object所有方法

1.getClass() 2.hashCode 3.clone 4.toString 5.equals 6.finalize 7.wait 8.notify 9.nofityAll 
当对象变成GC Roots不可达时，GC会判断对象是否重写finalize方法，未覆盖则回收。否则若对象未执行过finalize方法，将其放入F-Queue队列。finalize执行完后，GC会再次判断对象是否可达，不可达则回收。

### Sleep(),wait(),join(),yield()

1.sleep是Thread类的静态本地方法，wait是Object类的本地方法。
2.sleep方法不会释放锁Lock，但wait会释放，并且会加入等待队列中。
3.wait必须依赖synchronized关键字
4.sleep不需要唤醒，wait没指定时间需要被唤醒
5.sleep当前线程休眠，wait用于多线程之间的通信。
6.sleep会让出CPU执行时间且强制上下文切换，wait不一定，wait后还是有机会重新竞争到锁继续执行。

yield()执行后线程直接进入就绪状态，马上释放CPU执行权。
join()执行后线程进入阻塞状态，main线程等待T1执行完毕

```java
public static void main() {
	Thread t1 = new Thread();
  t1.start();
  t1.join();		//main线程等待T1执行完毕
}
```





### **内存分配方式：指针碰撞和空闲列表**

1.指针碰撞：如果Java堆中内存是绝对规整的，所有用过的内存都放在一边，空闲的内存放在另一边，中间放着一个指针作为分界点的指示器，那所分配内存就仅仅是把那个指针向空闲空间那边挪动一段与对象大小相等的距离，这种分配方式称为“指针碰撞”（Bump the Pointer）。

2.空闲列表：如果Java堆中的内存并不是规整的，已使用的内存和空闲的内存相互交错，那就没有办法简单地进行指针碰撞了，虚拟机就必须维护一个列表，记录上哪些内存块是可用的，在分配的时候从列表中找到一块足够大的空间划分对象实例，并更新列表上的记录，这种分配方式称为“空闲列表”（Free List）。

选择哪种分配方式由Java堆是否规整决定，而Java堆是否规整又由所采用的**垃圾收集器**是否带有**压缩整理**功能决定。
因此，在使用Serial、ParNew等带Compact过程的收集器时，系统采用的分配算法是指针碰撞，而使用CMS这种基于Mark-Sweep标记清除算法的收集器时，通常采用空闲列表。

### 内存分配的并发控制：CAS和TLAB

对象创建在虚拟机中是非常频繁的行为，即使是仅仅修改一个指针所指向的位置，在并发情况下也并不是线程安全的，可能出现正在给对象A分配内存，指针还没来得及修改，对象B又同时使用了原来的指针来分配内存的情况。

主要通过两种方式解决:
1.CAS加上失败重试分配内存地址。
2.TLAB 为每个线程分配一块缓冲区域进行对象分配，new对象内存的分配均需要进行加锁，这也是new开销比较大的原因，所以Sun Hotspot JVM为了提升对象内存分配的效率，对于所创建的线程都会分配一块独立的空间，这块空间又称为TLAB，TLAB仅作用于新生代的Eden，因此在编写Java程序时，通常多个小的对象比大的对象分配起来更加高效。使用-XX：+/-UseTLAB。

[内存分配文章](https://www.cnblogs.com/ZJOE80/p/12931989.html)













## 分布式问题

### 分布式ID生成方式

**1.UUID**
	**优点**： 唯一性   
	**缺点**：
		1.无序的字符串，不具备趋势自增特性
		2.没有具体的业务含义
		3.长度过长16 字节128位，36位长度的字符串，存储以及查询对MySQL的性能消耗较大，MySQL官方明确建议主键要尽量越短越好，作为数据库主键 `UUID` 的无序性会导致数据位置频繁变动，严重影响性能。
**2.数据库自增ID**
	优点：实现简单，ID单调自增，数值类型查询速度快
	**缺点**：DB单点存在宕机风险，无法扛住高并发场景
**3.数据库集群模式**
	**会生成重复的ID怎么办？**
	**解决方案**：设置`起始值`和`自增步长`

```bash
# MySQL_1 配置：
set @@auto_increment_offset = 1;     -- 起始值
set @@auto_increment_increment = 2;  -- 步长
# MySQL_2 配置：
set @@auto_increment_offset = 2;     -- 起始值
set @@auto_increment_increment = 2;  -- 步长
```

​	**优点：**解决DB单点问题
​	**缺点：**不利于后续扩容，而且实际上单个数据库自身压力还是大，依旧无法满足高并发场景。

**4.号段模式**

从数据库批量的获取自增ID，每次从数据库取出一个号段范围，例如 (1,1000] 代表1000个ID，具体的业务服务将本号段，生成1~1000的自增ID并加载到内存。

```mysql
CREATE TABLE id_generator (
  id int(10) NOT NULL,
  max_id bigint(20) NOT NULL COMMENT '当前最大id',
  step int(20) NOT NULL COMMENT '号段的布长',
  biz_type	int(20) NOT NULL COMMENT '业务类型',
  version int(20) NOT NULL COMMENT '版本号 保证并发时数据的正确性',
  PRIMARY KEY (`id`)
) 
# 取完更新表中数据   set max_id = #{max_id+step}, version = version + 1 where version = # {version} and biz_type = XXX
# 多业务端可能同时操作，所以采用版本号version乐观锁方式更新，这种分布式ID生成方式不强依赖于数据库，不会频繁的访问数据库，对数据库的压力小很多。
```

**5.Redis**

Redis也同样可以实现，原理就是利用redis的 incr命令实现ID的原子性自增。

```bash
127.0.0.1:6379> set seq_id 1     // 初始化自增ID为1
OK
127.0.0.1:6379> incr seq_id      // 增加1，并返回递增后的数值
(integer) 2
```

用`redis`实现需要注意一点，要考虑到redis持久化的问题。`redis`有两种持久化方式`RDB`和`AOF`

- `RDB`会定时打一个快照进行持久化，假如连续自增但`redis`没及时持久化，而这会Redis挂掉了，**重启Redis后会出现ID重复的情况**。
- `AOF`会对每条写命令进行持久化，即使`Redis`挂掉了也不会出现ID重复的情况，但由于incr命令的特殊性，会导致`Redis`重启恢复的数据时间过长。

**6.雪花算法（SnowFlake）**

`Snowflake`生成的是Long类型的ID，一个Long类型占8个字节，每个字节占8比特，也就是说一个Long类型占64个比特。
Snowflake ID组成结构：`正数位`（占1比特）+ `时间戳`（占41比特）+ `机器ID`（占5比特）+ `数据中心`（占5比特）+ `自增值`（占12比特），总共64比特组成的一个Long类型。

- 第一个bit位（1bit）：Java中long的最高位是符号位代表正负，正数是0，负数是1，一般生成ID都为正数，所以默认为0。
- 时间戳部分（41bit）：毫秒级的时间，不建议存当前时间戳，而是用（当前时间戳 - 固定开始时间戳）的差值，可以使产生的ID从更小的值开始；41位的时间戳可以使用69年，(1L << 41) / (1000L * 60 * 60 * 24 * 365) = 69年
- 工作机器id（10bit）：也被叫做`workId`，这个可以灵活配置，机房或者机器号组合都可以。
- 序列号部分（12bit），自增值支持同一毫秒内同一个节点可以生成4096个ID

可能出现机器时针回拨，导致重复ID

**7.Leaf segment 美团**



### 分布式锁

数据库：利用主键冲突控制一次只有一个线程获取锁
	问题：非阻塞、不可重入、单点、失效时间

Zookeeper分布式锁：
	zk通过临时节点，解决了死锁的问题，一旦客户端获取锁后突然挂掉，临时节点就是会删除掉，其他客户端自动获取锁，临时顺序节点解决惊群效应。

Redis分布式锁：setNX，单线程处理网络请求，不需要考虑并发安全性
	所有服务节点设置相同的key，返回为0，则锁获取失败

> setnx:
> 问题1.早期版本没有超时时间，expire设置，存在死锁问题 （中途机器下线）
> 问题2.后期版本提供锁和设置时间原子操作，但是存在任务重试，锁自动释放，导致并发问题，加锁和解锁不是同一个线程

可重入性和锁续期没有实现，通过redisson解决（类似AQS的实现，看门狗监听机制）

redlock



























## Git问答

patch可以通过邮件发送以及从邮件应用

git blame可以查看文件每一行的提交者

git grep可以搜索工作区文件，也可以搜索再暂存区注册过的blob对象。

git grep不能搜索指定tree对象中的blob对象内容 - X

如果希望执行git push的时候默认讲远程存在的分支所对应的本地分支都推送到远程，应该 git config --global push default matching

tree对象存储多个tree对象或者blob对象的信息

tag对象存储commit对象的信息

同一个blob对象的信息可以存储在不同的tree对象当中

查看本地哪些分支在远程库中已经不存在， git remote show origin

提交了一个有密码的文件，git filter -branch --tree-filter 'rm -f passwords.txt' HEAD

如果希望git commit弹出编辑框编辑提交信息完成提交，不要忽略提交信息中以#开头的注释行，但还是忽略头尾的空行，尾部的空格，应该git config --global commit cleanup whitespace
希望git能以中文命名路径和文件名，并且在跨平台上能正常显示，应该git config --global core.quotepath false
当前处于develop分支，执行git merge fixes enhancements是指合并fixes和enhancements分支到当前分支
Git remote add fork https://xxx 添加了一个远程仓库fork之后，执行git branch --set-upstream feature/foo fork/master，会报错，因为没有执行git fetch 获取fork远程仓库的信息。
要查看本地哪些分支在远程仓库已经不存在了，使用git remote show origin



### Ngix

什么是信号驱动IO模型：信号驱动IO是应用进程告诉内核：当你的数据报准备好的生活，给我发送一个信号，并且调用我的信号处理函数
每一个NginxWorke进程持有一个Lua解释器或者LuaJIT实例，被这个worker处理的全部请求共享这个实例。
ngx_lua将Lua嵌入Nginx，能够让Nginx运行lua脚本，并且高并发、非阻塞的处理各种请求。
ngx_lua在Lua中进行的IO操作都会托付给Nginx的事件模型，从而实现非阻塞调用。
Nginx反向代理配置proxy_buffers的使用，设置proxy_buffers缓存区。
nginx配置文件中，events是块配置项；块配置项可以嵌套；块配置项由一个块配置项名和一对大括号组成；





面试回归正题：
面试官问：项目中你对数据库做了什么优化？  
这个我会，自由发挥（比较简单） 

面试官问：对数据库分库分表了解吗？用什么算法进行分库分表 ？
分库分表我会，只会简单地操作，用什么算法不懂。 

https://zhuanlan.zhihu.com/p/84224499



面试官问：说一下mysql索引的原理  ？
这个简单我会，我从索引的数据结构分析。

面试官问：知道MySQL插入和查询分别用的是什么锁吗？
这个简单，大家都会

知道悲观锁吗？了解多少？ 
 这个最简单 

面试官问：说一下synchronized的优点和缺点，与lock进行比较？
 这个没有难度 

面试官问：说一下ReetrantLock的内部实现 ？
我看过ReetrantLock的源码，这个我会。

面试官问：说一下你对HashMap的理解？
 比较简单 ，我从hashmap的数据结构分析。

面试官问：现在有一亿条数据，要求你利用HashMap对数据进行去重并排序，你会怎么做？
 这个不会 ，有难度。


​    
面试官问：Redisson分布式锁？
这个我会，我在上家公司用过，在高并发用分布式锁防止订单Id重复。

面试官问：Netty高性能表现在哪些方面？
这个我会，回答的范围比较广。

面试官问：maven如果你的项目里有两个包的版本冲突了 你是怎么解决的？
我说：用 exclusion 标签排除掉不需要的 jar 包。

面试官问：JAVA实现多态的原理？
不会。（平时只是简单用一下）
面试官问：RPC的原理？
不懂，上次也是这个面试题。

面试官问：https原理？
不会
    
面试官问：HashMap在并发环境下会出现什么样的问题？
这个不会，我上家公司所有的数据用pojo封装返回，没有用hashmap，这个方面不是了解很深。  


面试官问：项目问题，主要是问我项目你做的业务。（这个自由发挥，原因是上家公司项目是你负责，所有面试题，最简单就是这个面试题）

还有很多问题不会回答。


三面总体来说，还是比较简单，面试官没有为难我，只是我不会，面试过程暗中提醒我，但是还是不会，这次面试回答个75%左右。

我跟面试官说：这次面试你对我有什么评价，他说，我基础还是可以，业务代码基本上没有问题，他说我虽然工作了5年，能力达不到5年开发经验的水平，你的能力顶多在3年开发经验的水平。











## 1. synchronized 关键字底层是如何实现的？它与 Lock 相比优缺点分别是什么？

1.synchronized不需要自己解锁, lock需要有unlock操作

2.JDK1.6前synchronize是重量锁，无法获取锁就挂起线程，1.6后给锁增加了四种状态，无锁状态 -> 偏向锁 -> 轻量级锁 -> 重量级锁

**synchronized总结：**

|  锁状态  |             假定情况             |       竞争处理策略       |
| :------: | :------------------------------: | :----------------------: |
|  偏向锁  |  假定获取锁的一直都是同一个线程  |       升级为轻量锁       |
| 轻量级锁 | 假定锁被占用时不会有其他线程获取 |    自旋等待，超时升级    |
| 重量级锁 |      最坏情况，经常发生竞争      | 直接将要获取锁的线程挂起 |

**最后的比较总结**

|     .      |       synchronized       |              lock               |
| :--------: | :----------------------: | :-----------------------------: |
|   1.使用   |      不需要自己解锁      |            手动释放             |
| 2.加锁方式 | 将普通对象与类对象做为锁 |    专门的 Lock 类对象作为锁     |
|   3.中断   |         不能响应         |           可响应中断            |
| 4.竞争解决 |    一系列的锁膨胀策略    |          加入排队队列           |
|  5.公平性  |          非公平          |    公平/非公平 两种模式可选     |
|  6.锁类型  |          悲观锁          | 乐观锁（基于volatile和cas实现） |

**底层实现**

1、synchronznized映射成字节码指令就是增加两个指令：monitorenter、monitorexit，

当一条线程执行时遇到monitorenter指令时，它会尝试去获得锁，如果获得锁，那么所计数器+1（为什么要加1，因为它是可重入锁，可根据这个琐计数器判断锁状态），如果没有获得锁，那么阻塞，当它遇到一个monitoerexit时，琐计数器会-1，当计数器为0时，就释放锁（tips：节码中出现的两个monitoerexit指令的原因是：一个正常执行-1，令一个异常时执行，这两个用goto的方式只执行一个）

2、Lock底层则基于volatile和cas实现



synchronized是一种互斥锁，一次只允许一个线程进入被锁住的代码块。

### 1.1 锁的对象：

1. 修饰的实例方法，对应的锁是对象实例
2. 修饰的是静态方法，对应的锁是当前类的Class实例
3. 修饰的是代码块，对应的锁是传入syn的对象实例

### 1.2 synchronized的原理

通过反编译可以发现，当修饰方法时，编译器会生成ACC_SYNCHRONIZED关键字用来标识。当修饰代码块时，会依赖monitorenter和monitorexit指令。

在内存中，对象一般由3个部分组成，对象头、对象实际数据和对齐填充。对象头Mark Word会纪录对象关于锁的信息。每个对象都会有一个与之对应的monitor对象，monitor对象中存储着当前持有锁的线程以及等待锁的线程队列。

### 1.3 JDK1.6后的优化

JDK1.6之前是重量级锁，线程进入同步代码块、方法时，monitor对象就会把当前进入线程的id进行存储，设置Mark Word对象地址，并把阻塞的线程存储到monitor的等待线程队列中。加锁是依赖底层操作系统的mutex相关指令实现，所以会有用户态和内核态之间的切换，性能损耗十分明显。

JDK1.6之后引入偏向锁和轻量级锁在JVM层面实现加锁的逻辑，不依赖底层操作系统，就没有切换的消耗。所以，Mark Word对锁的状态纪录一共有4种：无锁、偏向锁、轻量级锁和重量级锁。

偏向锁：JVM认为只有某个线程才会执行同步代码，没有竞争的环境。Mark Word会直接纪录线程ID，只要线程来执行代码，会对比线程ID是否相等，相等则当前线程能直接获取得到锁，执行同步代码。如果不相等，则用CAS来尝试修改当前的线程ID，如果CAS修改成功，那还是能获取得到锁，执行同步代码。如果CAS失败了，说明有竞争环境，此时会撤销偏向锁升级成轻量级锁。

轻量级锁：当前线程会在栈帧下创建Lock Record，Lock Record会把Mark Word的信息拷贝进去，且有个Owner指针指向加锁的对象。线程执行到同步代码块时，则用CAS试图将Mark Word的指向到线程栈帧的Lock Record,假设CAS修改成功，则获取得到轻量级锁。修改失败，重试一定次数，升级成为重量级锁。

### 1.4总结

synchronized锁原来只有重量级锁，依赖操作系统的mutex指令，需要用户态和内核态切换，性能消耗明显。重量级锁用到monitor对象，而偏向锁则在Mark Word纪录线程ID进行对比。轻量级锁则是拷贝Mark Word到Lock Record，用CAS+自旋的方式获取。锁只有升级没有降级。1、只有一个线程进入临界区，偏向锁。2、多个线程交替进入临界区，轻量级锁。3、多线程同时进入临界区，重量级锁。





##  2. Java 中垃圾回收机制中如何判断对象需要回收？常见的 GC 回收算法有哪些？

ParNew GC

CMS (Concurrent Mark Sweep)
	基于标记-清除 Mark-Sweep
	设计目标尽量减少停顿时间
	内存碎片化问题
	CMS会占用更多CPU资源，和用户线程争抢

Parallel GC
	JDK8中的默认GC
	新生代和老年代GC是并行进行的
	-XX:+UseParallelGC

G1 GC

ZGC 
	JDK11中新引入低延迟垃圾回收器



GC调优思路

1. 如果服务器一次卡顿时间比较长，一般是full gc时间过长，而应用追求的是卡顿（STW）时间短，可以接受多次卡顿，那么可以考虑调整加大young gen的比例，和提高进入老年代的年龄（默认15）来尽量避免FGC
2. 选择合适的收集器很重要。要根据应用的特点。是追求吞吐量，还是追求最小停顿。	
3. 经常对照gc日志观察现实的情况，如多久时间一卡顿，来调整自己的收集器和相关的内存比例
4. 在有限的物理资源条件下，要避免让用户接受过多的STW，可以考虑在半夜自动进行gc（System.gc()），虽然不一定生效，但可以观察下情况。多数情况下是会出发full gc的
5. 大多数应用是可以接受频繁的mgc，但却不能接受full gc的长时间卡顿，所以在观察gc日志时一定要注意自己full gc的频率和触发条件（是由于内存担保，还是年龄到了，还是TO内存太小，导致每次都fgc）







## 3. SpringBoot 是如何进行自动配置的？

**什么是SpringBoot自动配置？**
 springboot的自动配置，指的是springboot会自动将一些配置类的bean注册进ioc容器，我们可以需要的地方使用@autowired或者@resource等注解来使用它。
 “自动”的表现形式就是我们只需要引我们想用功能的包，相关的配置我们完全不用管，springboot会自动注入这些配置bean，我们直接使用这些bean即可。



## 4. 简述 JVM 的内存模型 JVM 内存是如何对应到操作系统内存的？

java的虚拟机种有两种线程，一种叫叫守护线程，一种叫非守护线程，main函数就是个非守护线程，虚拟机的gc就是一个守护线程。

java虚拟机的生命周期，当一个java应用main函数启动时虚拟机也同时被启动，而只有当在虚拟机实例中的**所有非守护进程都结束时**，java虚拟机实例才结束生命。

![img](../../images/1626845-20190424211439503-1707501469.png)

线程私有的：程序计数器，虚拟机栈，本地方法栈
线程共享的：堆，方法区，直接内存

1.**程序计数器**：是唯一不会出现 OutOfMemoryError 的内存区域，它的生命周期随着线程的创建而创建，随着线程的结束而死亡。

2.**虚拟机栈**：虚拟机栈是由一个个栈帧组成，线程在执行一个方法时，便会向栈中放入一个栈帧，每个栈帧中都拥有局部变量表、操作数栈、动态链接、方法出口信息。局部变量表存放基本数据类型（boolean、byte、char、short、int、float、long、double）和对象引用（reference)。
虚拟机栈会出现两种异常：StackOverFlowError 和 OutOfMemoryError。
	StackOverFlowError：若Java虚拟机栈的内存大小不允许动态扩展，那么当线程请求栈的深度超过当前Java虚拟机栈的最大深度的时候，就抛出StackOverFlowError异常。
	OutOfMemoryError：若 Java 虚拟机栈的内存大小允许动态扩展，且当线程请求栈时内存用完了，无法再动态扩展了，此时抛出OutOfMemoryError异常。

3.本地方法栈：本地方法栈则为虚拟机使用到的 Native 方法服务

4.堆是Java 虚拟机所管理的内存中最大的一块，几乎所有的对象实例以及数组都在这里分配内存。也被称作GC堆，收集器基本都采用分代垃圾收集算法。“分代回收”是基于这样一个事实：对象的生命周期不同，所以针对不同生命周期的对象可以采取不同的回收方式，以便提高回收效率。

5.方法区:它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。

6.运行时常量池：运行时常量池是方法区的一部分。Class 文件中除了有类的版本、字段、方法、接口等描述信息外，还有常量池信息（用于存放编译期生成的各种字面量和符号引用）
既然运行时常量池时方法区的一部分，自然受到方法区内存的限制，当常量池无法再申请到内存时会抛出 OutOfMemoryError 异常。



## 5.String 类能不能被继承？为什么？

不可以，因为String类有final修饰符，而final修饰的类是不能被继承的

```java
public final class String implements java.io.Serializable, Comparable<String>, CharSequence String s = new String(“xyz”);  //创建了几个对象
答案： 1个或2个， 如果”xyz”已经存在于常量池中，则只在堆中创建”xyz”对象的一个拷贝，否则还要在常量池中在创建一份
  
String为immutable, 不可更改的，每次String对象做累加时都会创建StringBuilder对象， 效率低下。
String s1 = "a";
String s2 = "b"; 
String s3 = s1 + s2; // 等效于 String s3 = (new StringBuilder(s1)).append(s2).toString();
```

设计成final的好处：

1. 不可变线程安全
2. 字符串常量池提升性能，修改数据是在内存重新开内存
3. String在Map中做key保证HashCode不变



## 6.请解释字符串比较之中“==”和equals()的区别？

-  ==：比较的是两个字符串内存地址（堆内存）的数值是否相等，属于数值比较；
-  equals()：比较的是两个字符串的内容，属于内容比较。



## 7.分代回收的时候，用了什么算法？

1. 复制算法

2. 标记-清除算法（Mark-Sweep）

   扫描所有对象标记出需要回收的对象，在标记完成后扫描回收所有被标记的对象，所以需要扫描两遍。
   问题：回收效率略低；标记清除之后会产生大量不连续的内存碎片，空间碎片太多可能会导致以后在程序运行过程中需要分配较大对象时，无法找到足够的连 续内存而不得不提前触发另一次垃圾回收动作

3. 标记-整理算法（Mark-Compact）

   首先标记出所有需要回收的对象，在标记完成后，后续步骤不是直接对可回收对象进行清理，而是让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存。

   问题：效率偏低；没有内存碎片；标记整理与标记清除算法的区别主要在于对象的移动。对象移动不单单会加重系统负担，同时需要全程暂停用户线程才能进行，同时所有引用 对象的地方都需要更新（直接指针需要调整）

| 回收器            | 回收对象和算法                        | 回收器类型         |
| ----------------- | ------------------------------------- | ------------------ |
| Serial            | 新生代，复制算法                      | 单线程(串行)       |
| Serial Old        | 老年代，标记整理算法                  | 单线程(串行)       |
| Parallel Scavenge | 新生代，复制算法                      | 并行的多线程收集器 |
| Parallel Old      | 老年代，标记整理算法                  | 并行的多线程回收器 |
| ParNew            | 新生代，复制算法                      | 并行的多线程收集器 |
| CMS               | 老年代，标记清除算法                  | 并发的多线程回收器 |
| G1                | 跨新生代和老年代；标记整理 + 化整为零 | 并发的多线程回收器 |



## 8.讲下类加载的过程？

当程序要使用某个类时，如果该类还未被加载到内存中，则系统会通过加载，连接，初始化三步来实现这个类进行初始化。

1. 加载

   加载，是指Java虚拟机查找字节流（查找.class文件），并且根据字节流创建java.lang.Class对象的过程。

   系统会创建唯一的Class对象，这个Class对象描述了这个类创建出来的对象的所有信息

2. 链接

   （1）验证阶段。主要的目的是确保被加载的类（.class文件的字节流）满足Java虚拟机规范，不会造成安全错误。

   （2）准备阶段。负责为类的静态成员分配内存，并设置默认初始值。

   （3）解析阶段。将类的二进制数据中的符号引用替换为直接引用。

3. 初始化

   只对static修饰的变量或语句块进行初始化。

   如果初始化一个类的时候，其父类尚未初始化，则优先初始化其父类。

   如果同时包含多个静态变量和静态代码块，则按照自上而下的顺序依次执行。



## 9.springboot自动配置的原理？

**配置属性**

```java
@ConfigurationProperties(prefix = "test")
public class Demo {
  private String msg="default";
}
// application.yml中配置信息
test:
  msg: bamboo
```

















## 10.spring如何解决循环依赖？

什么是循环依赖? 简单的说就是A依赖B，B依赖C，C依赖A这样就构成了循环依赖。

Spring解决循环依赖的方法就是如题所述的三级缓存、预曝光。Spring的三级缓存主要是singletonObjects、earlySingletonObjects、singletonFactories这三个Map：

```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    Object singletonObject = this.singletonObjects.get(beanName);　　//首先通过beanName从一级缓存获取bean
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {　　//如果一级缓存中没有，并且beanName映射的bean正在创建中
        synchronized (this.singletonObjects) {
            singletonObject = this.earlySingletonObjects.get(beanName);　　//从二级缓存中获取
            if (singletonObject == null && allowEarlyReference) {　　//二级缓存也没有
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);　　//从三级缓存获取
                if (singletonFactory != null) {
                    singletonObject = singletonFactory.getObject();　　//获取到bean
                    this.earlySingletonObjects.put(beanName, singletonObject);　　//将获取的bean提升至二级缓存
                    this.singletonFactories.remove(beanName);　　//从三级缓存删除
                }
            }
        }
    }
    return singletonObject;
}
```

###  为何需要三级缓存解决循环依赖，而不是二级缓存 https://www.cnblogs.com/semi-sub/p/13548479.html



### 谈谈你对Java平台的理解？“Java是解释执行”，这句话正确吗？

我们开发的Java的源代码，首先通过Javac编译成为字节码（bytecode），然后，在运行时，通过 Java虚拟机（JVM）内嵌的解释器将字节码转换成为最终的机器码。



### synchronized和ReentrantLock有什么区别？有人说synchronized最慢，这话靠谱吗？- 极客15

1. 理解什么是线程安全。
2. synchronized、ReentrantLock等机制的基本使用与案例。
3. 掌握synchronized、ReentrantLock底层实现；理解锁膨胀、降级；理解偏斜锁、自旋锁、轻量级锁、重量级锁等概念。
4. 掌握并发包中java.util.concurrent.lock各种不同实现和案例分析。



## 11. CountDownLatch

### 11.1CountDownLatch的用法

CountDownLatch典型用法：1、某一线程在开始运行前等待n个线程执行完毕。将CountDownLatch的计数器初始化为new CountDownLatch(n)，每当一个任务线程执行完毕，就将计数器减1 countdownLatch.countDown()，当计数器的值变为0时，在CountDownLatch上await()的线程就会被唤醒。一个典型应用场景就是启动一个服务时，主线程需要等待多个组件加载完毕，之后再继续执行。

CountDownLatch典型用法：2、实现多个线程开始执行任务的最大并行性。注意是并行性，不是并发，强调的是多个线程在某一时刻同时开始执行。类似于赛跑，将多个线程放到起点，等待发令枪响，然后同时开跑。做法是初始化一个共享的CountDownLatch(1)，将其计算器初始化为1，多个线程在开始执行任务前首先countdownlatch.await()，当主线程调用countDown()时，计数器变为0，多个线程同时被唤醒。

### 11.2**CountDownLatch**的不足

CountDownLatch是一次性的，计算器的值只能在构造方法中初始化一次，之后没有任何机制再次对其设置值，当CountDownLatch使用完毕后，它不能再次被使用。



## 12.如何优雅中断线程

### 12.1调用stop、resume、suspend方法，不会释放锁持有的锁，容易出现死锁

### 12.2使用interrupt中断线程

interrupt(): 用于线程中断，该方法并不能直接中断线程，只会将线程的中断标志位改为true。它只会给线程发送一个中断状态，线程是否中断取决于线程内部对该中断信号做什么响应，若不处理该中断信号，线程就不会中断。
interrupted(): 判断线程是否处于中断状态，该方法调用后会将线程的中断标志位重置为false。
isInterrupted(): 判断线程是否处于中断状态。

```java
public class ThreadTest implements Runnable{

    @Override
    public void run() {
        // 在线程体中对线程的中断标志位进行判断，若线程中断，则不再执行
        while (!Thread.currentThread().isInterrupted()){
            System.out.println("Thread is running");

            try {
                System.out.println(Thread.currentThread().getName() + " " + Thread.currentThread().isInterrupted());
                Thread.sleep(100);
            }
            /**
             * 需要注意的是，当方法体中的代码抛出InterruptedException异常时，线程的中断标志位会复位成          
             * false，若不处理，外部中断线程时，内部也无法停止，所以在catch代码中手动处理，将线程中断
             */
            catch (InterruptedException e) {
                System.out.println(Thread.currentThread().getName() + ":" + Thread.currentThread().isInterrupted());
                // 发生InterruptedException异常时，在catch中处理，中断线程
                Thread.currentThread().interrupt();
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + ":" + Thread.currentThread().isInterrupted());
        }
    }
}
```





CPU每个周期做什么事情？

反射的缺点是什么？如何优化？

优点： 运行期类型的判断，动态加载类，提高代码灵活度。
缺点： 性能瓶颈：反射相当于一系列解释操作，通知 JVM 要做的事情，性能比直接的java代码要慢很多。







## 13.Semaphore原理浅析和相关面试题分析

问题1.semaphore初始化有10个令牌，11个线程同时各调用1次acquire方法，会发生什么？
	答案：拿不到令牌的线程阻塞，不会继续往下运行。

问题2.semaphore初始化有10个令牌，一个线程重复调用11次acquire方法，会发生什么？
	答案：线程阻塞，不会继续往下运行。可能你会考虑类似于锁的重入的问题，很好，但是，令牌没有重入的概念。你只要调用一次acquire方法，就需要有一个令牌才能继续运行。

问题3.semaphore初始化有1个令牌，1个线程调用一次acquire方法，然后调用两次release方法，之后另外一个线程调用acquire(2)方法，此线程能够获取到足够的令牌并继续运行吗？
	答案：能，原因是release方法会添加令牌，并不会以初始化的大小为准。

问题4.semaphore初始化有2个令牌，一个线程调用1次release方法，然后一次性获取3个令牌，会获取到吗？
	答案：能，原因是release会添加令牌，并不会以初始化的大小为准。Semaphore中release方法的调用并没有限制要在acquire后调用。

```java
 public static void main(String[] args) {
 	int permitsNum = 2;
  final Semaphore semaphore = new Semaphore(permitsNum);
  System.out.println("permits:"+semaphore.availablePermits()+
      ",tryAcquire(3,1,TimeUnit.SECONDS):"+semaphore.tryAcquire(3,1, TimeUnit.SECONDS));
  semaphore.release();
  System.out.println("permits:"+semaphore.availablePermits()+
      ",tryAcquire(3,1, TimeUnit.SECONDS):"+semaphore.tryAcquire(3,1, TimeUnit.SECONDS));
}

permits:2,tryAcquire(3,1, TimeUnit.SECONDS):false
permits:3,tryAcquire(3,1, TimeUnit.SECONDS):true
```



## 14.Java 线程和操作系统的线程是怎么对应的？Java线程是怎样进行调度的?

每个继承java.lang.Thread的类，调用start方法之后，都调用start0()的native方法;start0()的native方法在openjdk里调用的是JVM_StartThread;JVM_StartThread最终调用的是操作系统的pthread_create()函数,有四个参数,我们写的run方法就是该函数的第三个参数.

## 15.简述新生代与老年代的区别？

新生代又分为Eden和Survivor两个区。数据会首先分配到Eden区当中（当然也有特殊情况，如果是大对象(需要大量连续内存空间的java对象)那么会直接放入到老年代。对象在Survivor每熬过一次Minor GC，年龄就加1，当年龄达到一定的程度（默认为15）时，就会被晋升到老年代中。

Java对象包含：对象头、对象体和对齐字节。



## 16.@Component，@Service等注解是如何被解析的？

浏览ContextNamespaceHandler

```java
springframework/context/annotation/ClassPathBeanDefinitionScanner.class:105
Set<BeanDefinitionHolder> doScan(String... basePackages) 
  // 读资源转换为BeanDefinition
  this.findCandidateComponents(basePackage); 
// 从classPath扫描组件，并转换为备选BeanDefinition，也就是要做的解析@Component的核心方法。

public Set<BeanDefinition> findCandidateComponents(String basePackage) {
		this.addCandidateComponentsFromIndex(this.componentsIndex, basePackage);
    this.scanCandidateComponents(basePackage);
}

public class ClassPathScanningCandidateComponentProvider implements EnvironmentCapable, ResourceLoaderAware
private Set<BeanDefinition> scanCandidateComponents(String basePackage)
// this.resourcePattern; 将package转化为ClassLoader类资源搜索路径packageSearchPath，例如：com.wl.spring.boot转化为classpath*:com/wl/spring/boot/**/*.class
String packageSearchPath = "classpath*:" + this.resolveBasePackage(basePackage)+"/**/*.class";
// 加载搜素路径下的资源。 
Resource[] resources = getResourcePatternResolver().getResources(packageSearchPath); 
for (Resource resource : resources) {
	//省略部分代码
  if (resource.isReadable()) {
     for (Resource resource : resources) {
     	if (resource.isReadable()) {
        MetadataReader metadataReader = getMetadataReaderFactory().getMetadataReader(resource);
      	if (isCandidateComponent(sbd)) {	// 判断是否是备选组件
          ScannedGenericBeanDefinition sbd = new ScannedGenericBeanDefinition(metadataReader);
        	sbd.setSource(resource);
        	candidates.add(sbd);					// 添加到返回结果的list
        }
      }
     }
  }
```

@Component是任何Spring管理的组件的通用原型。@Repository、@Service和@Controller是派生自@Component。

```java
public @interface Service {
   @AliasFor(annotation = Component.class)
   String value() default "";
}
```

@Component是@Service的元注解

**总结**

**1.将package转化为ClassLoader类资源搜索路径packageSearchPath**

**2.加载搜素路径下的资源。**

**3.isCandidateComponent 判断是否是备选组件。**

内部调用的TypeFilter的match方法：

- `AnnotationTypeFilter#matchself中metadata.hasMetaAnnotation`处理元注解
- `metadata.hasMetaAnnotation=AnnotationMetadataReadingVisitor#hasMetaAnnotation`

就是判断当前注解的元注解在不在metaAnnotationMap中。

AnnotationAttributesReadingVisitor#visitEnd()内部方法recursivelyCollectMetaAnnotations 递归的读取注解，与注解的元注解（读@Service，再读元注解@Component），并设置到metaAnnotationMap

**4.添加到返回结果的list**



## 17.Autowired Service Component Bean

@Autowired是Spring的注解，Autowired默认先按byType，如果发现找到多个bean，则又按照byName方式比对，如果还有多个，则报出异常；
@Resource 是JDK1.6支持的注解，默认按照名称(Byname)进行装配, 如果没有指定name属性，当注解写在字段上时，默认取字段名，按照名称查找，如果注解写在setter方法上默认取属性名进行装配。当找不到与名称匹配的bean时才按照类型进行装配。但是需要注意的是，如果name属性一旦指定，就只会按照名称进行装配。
不过推荐使用@Resource一点，因为当接口有多个实现时@Resource直接就能通过name属性来指定实现类，而@Autowired还要结合@Qualifier注解来使用，且@Resource是jdk的注释，可与Spring解耦。

在SpringBoot中当我们要声明一个Bean的时候，我们可以在该类上加上 **@Service，@Compont** 等，或者是在配置类中加上 **@Bean** 这个注解，除此之外还有一种方法，就是 **@Import**



















### java的基本数据类型和字节数

### mysqI索引结构,特点，为什么使用这个

### 聚集索引和非聚集索引



String，StringBuffer, StringBuilder区别

HashMap，为什么使用红黑树

子类和父类的实例变量和方法有什么区别

java泛型

悲观锁和乐观锁

mysql底层原理，为什么效率高，主键能不能太大，为什么

linux查询tcp连接处理CLOSE_ WAIT的状态的数目

RabbitMQ, kafka, RocketMQ, ActiveMQ, 以及其他消息中间件

redis为什么效率高，线程，数据结构,网络模型，aio, nio, bio, 为什么这么设计？如何处理高并发

分布系统的设计，分布式系统CAP，分布式系统的模型

linux环境下的线上业务管理有没有，如何管理

redis的集合有没有限制，限制是多少

redis的1w条的插入和更新有什么区别

mysql join的底层原理是什么，有哪几种(不是左右连接这种)

linux命令查询一个文件内出现重复最多的数字的

linux命令查询一个文件的行数



## 18.自定义注解之运行时注解(RetentionPolicy.RUNTIME)

对注解概念不了解的可以先看这个：[Java注解基础概念总结](http://blog.csdn.net/github_35180164/article/details/52107204)

前面有提到注解按生命周期来划分可分为3类：

1、RetentionPolicy.SOURCE：注解只保留在源文件，当Java文件编译成class文件的时候，注解被遗弃；
2、RetentionPolicy.CLASS：注解被保留到class文件，但jvm加载class文件时候被遗弃，这是默认的生命周期；
3、RetentionPolicy.RUNTIME：注解不仅被保存到class文件中，jvm加载class文件之后，仍然存在；

这3个生命周期分别对应于：Java源文件(.java文件) ---> .class文件 ---> 内存中的字节码。





## 19.Java注解基础概念总结

**注解的概念**

注解（Annotation），也叫元数据（Metadata），是Java5的新特性，JDK5引入了Metadata很容易的就能够调用Annotations。注解与类、接口、枚举在同一个层次，并可以应用于包、类型、构造方法、方法、成员变量、参数、本地变量的声明中，用来对这些元素进行说明注释。

**注解的语法与定义形式**

（1）以@interface关键字定义
（2）注解包含成员，成员以无参数的方法的形式被声明。其方法名和返回值定义了该成员的名字和类型。
（3）成员赋值是通过@Annotation(name=value)的形式。
（4）注解需要标明注解的生命周期，注解的修饰目标等信息，这些信息是通过元注解实现。

以 java.lang.annotation 中定义的 Target 注解来说明：

```java
@Retention(value = RetentionPolicy.RUNTIME)
@Target(value = { ElementType.ANNOTATION_TYPE } )
public @interface Target {
    ElementType[] value();
}
```

**源码分析如下：**
第一：元注解@Retention，成员value的值为RetentionPolicy.RUNTIME。
第二：元注解@Target，成员value是个数组，用{}形式赋值，值为ElementType.ANNOTATION_TYPE
第三：成员名称为value，类型为ElementType[]
另外，需要注意一下，如果成员名称是value，在赋值过程中可以简写。如果成员类型为数组，但是只赋值一个元素，则也可以简写。如上面的简写形式为：

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Target {
    ElementType[] value();
}
```

**注解的分类有两种分法：**

第一种分法

1、基本内置注解，是指Java自带的几个Annotation，如@Override、Deprecated、@SuppressWarnings等；

2、元注解（meta-annotation），是指负责注解其他注解的注解，JDK 1.5及以后版本定义了4个标准的元注解类型，如下：

```java
@Target
@Retention
@Documented
@Inherited
```

3、自定义注解，根据需要可以自定义注解，自定义注解需要用到上面的meta-annotation

第二种分法

注解需要标明注解的生命周期，这些信息是通过元注解 @Retention 实现，注解的值是 enum 类型的 RetentionPolicy，包括以下几种情况：

```java
public enum RetentionPolicy {
    /**
     * 注解只保留在源文件，当Java文件编译成class文件的时候，注解被遗弃.
     * 这意味着：Annotation仅存在于编译器处理期间，编译器处理完之后，该Annotation就没用了
     */
    SOURCE,	
    /**
     * 注解被保留到class文件，但jvm加载class文件时候被遗弃，这是默认的生命周期.
     */
    CLASS,
    /**
     * 注解不仅被保存到class文件中，jvm加载class文件之后，仍然存在，
     * 保存到class对象中，可以通过反射来获取
     */
    RUNTIME
}
```

元注解

如上所介绍的Java定义了4个标准的元注解：

```java
// @Documented：标记注解，用于描述其它类型的注解应该被作为被标注的程序成员的公共API，因此可以被例如javadoc此类的工具文档化。
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Documented {
}

// @Inherited：标记注解，允许子类继承父类的注解
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Inherited {
}

// @Retention：指Annotation被保留的时间长短，标明注解的生命周期，3种RetentionPolicy取值含义上面已说明
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Retention {
    RetentionPolicy value();
}

// @Target：标明注解的修饰目标，共有
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Target {
    ElementType[] value();
}

// ElementType取值
public enum ElementType {
    /** 类、接口（包括注解类型）或枚举 */
    TYPE,
    /** field属性，也包括enum常量使用的注解 */
    FIELD,
    /** 方法 */
    METHOD,
    /** 参数 */
    PARAMETER,
    /** 构造函数 */
    CONSTRUCTOR,
    /** 局部变量 */
    LOCAL_VARIABLE,
    /** 注解上使用的元注解 */
    ANNOTATION_TYPE,
    /** 包 */
    PACKAGE
}
```





## ThreadLocal

Threadlocal主要用来线程变量的隔离

```java
class ThreadLocal {
  public void set(T value) {￼   
    Thread t = Thread.currentThread();￼
    ThreadLocalMap map = getMap(t);￼
    if(map ! = null)￼
      map.set(this, value);￼
    else￼
      createMap(t, value);￼
  }
  ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
  }
  void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
  }
  
  public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
      ThreadLocalMap.Entry e = map.getEntry(this);
      if (e != null)
        return (T)e.value;
    }
    return setInitialValue(); 	// 返回默认值
  }
}

class ThreadLocalMap {
  WeakReference<ThreadLocal>;		// 便于垃圾回收
}

struct thread {		// 线程结构
  ThreadLocalMap threadLocals;
}
```

1）ThreadLocal使得每个线程对同一个变量有自己的独立副本，是实现线程安全、减少竞争的一种方案。
2）ThreadLocal经常用于存储上下文信息，避免在不同代码间来回传递，简化代码。
3）每个线程都有一个Map，调用ThreadLocal对象的get/set实际就是以ThreadLocal对象为键读写当前线程的该Map。





## 20.缓存

**缓存穿透**缓存穿透是指缓存和数据库中都没有的数据，而用户不断发起请求，如发起为id为“-1”的数据或id为特别大不存在的数据。这时的用户很可能是攻击者，攻击会导致数据库压力过大。

 **解决方案：**
    1.接口层增加校验，如用户鉴权校验，id做基础校验，id<=0的直接拦截；
    2.从缓存取不到的数据，在数据库中也没有取到，这时也可以将key-value对写为key-null，缓存有效时间可以设置短点，如30秒（设置太长会导致正常情况也没法使用）。这样可以防止攻击用户反复用同一个id暴力攻击

**缓存击穿**缓存击穿是指缓存中没有但数据库中有的数据（一般是缓存时间到期），这时由于并发用户特别多，同时读缓存没读到数据，又同时去数据库去取数据，引起数据库压力瞬间增大，造成过大压力

**解决方案：**
	1.设置热点数据永远不过期。
    2.加互斥锁，互斥锁参考代码如下：

```java
public static String getData(String key) {
	String result = getDataFromRedis(key);  // REDIS
  if (result == null) {
    if (reenLock.tryLock()) {		// 加锁
      result = getDataFromMysql(key);		// DB
      if (result != null)
        setDataToCache(key, result);
      reenLock.unlock();		// 释放锁
    } else {		// 未获取锁的线程
      Thread.sleep(100);
      result = getData(key);	// 防止都去数据库重复取数据
    } 
  }
  return result;
}
```

**缓存雪崩**缓存雪崩是指缓存中数据大批量到过期时间，而查询数据量巨大，引起数据库压力过大甚至down机。和缓存击穿不同的是，    缓存击穿指并发查同一条数据，缓存雪崩是不同数据都过期了，很多数据都查不到从而查数据库。

**解决方案**：

1. 缓存数据的过期时间设置随机，防止同一时间大量数据过期现象发生。
2. 如果缓存数据库是分布式部署，将热点数据均匀分布在不同搞得缓存数据库中。
3. 设置热点数据永远不过期。













# REDIS

### 1.Redis 为什么把简单的字符串设计成 SDS？

SDS结构

```
struct sdshdr{
  int free; // buf[]数组未使用字节的数量
  int len; // buf[]数组所保存的字符串的长度
  char buf[]; // 保存字符串的数组
}
```

1.效率高   C字符串，在获取一个字符串长度时，需对整个字符串进行遍历，此时的复杂度是O(N)。

## 

### 2.Redis 过期策略:

   **定期删除+惰性删除**  **走内存淘汰机制**

- **allkeys-lru**：当内存不足以容纳新写入数据时，在**键空间**中，移除最近最少使用的 key（这个是**最常用**的）。
- volatile-lru：当内存不足以容纳新写入数据时，在**设置了过期时间的键空间**中，移除最近最少使用的 key（这个一般不太合适）。
- volatile-random：当内存不足以容纳新写入数据时，在**设置了过期时间的键空间**中，**随机移除**某个 key。
- volatile-ttl：当内存不足以容纳新写入数据时，在**设置了过期时间的键空间**中，有**更早过期时间**的 key 优先移除。

### 3.如何保证 redis 的高并发和高可用？redis 的主从复制原理能介绍一下么？redis 的哨兵原理能介绍一下么？

- [redis 主从架构](https://doocs.github.io/advanced-java/#/docs/high-concurrency/redis-master-slave)
- [redis 基于哨兵实现高可用](https://doocs.github.io/advanced-java/#/docs/high-concurrency/redis-sentinel)

redis 实现**高并发**主要依靠**主从架构**，一主多从，一般来说，很多项目其实就足够了，单主用来写入数据，单机几万 QPS，多从用来查询数据，多个从实例可以提供每秒 10w 的 QPS。

如果想要在实现高并发的同时，容纳大量的数据，那么就需要 redis 集群，使用 redis 集群之后，可以提供每秒几十万的读写并发。

redis 高可用，如果是做主从架构部署，那么加上哨兵就可以了，就可以实现，任何一个实例宕机，可以进行主备切换。





面试官问：假如Redis里面有1亿个key，其中有10w个key是以某个固定的已知的前缀开头的，如果将它们全部找出来？
 这个简单   

面试官问：redis底层实现原理？
这个面试题太难了。完全不会

面试官问：Redis集群原理？ 
这个比较简单   ，我从redis主从，哨兵架构   ，插槽上分析。     

面试官问：redis主从复制的工作原理能说一下么？
这个我会。（看书学过）



redis实现分布式锁有什么劣势，和其他分布式锁有什么优劣势

**分布式锁实现方式**
1.基于数据库实现分布式锁； 
2.基于缓存（Redis等）实现分布式锁； 
3.基于Zookeeper实现分布式锁；







Redis 作为消息队列为什么不能保证 100% 的可靠性？ 







# MySQL

4种隔离级别：
	1.读未提交  Read Uncommitted  - 脏读
	2.读提交 Read Commited  - 不可重复读
	3.可重复读 Repeatable Read - 
	4.串行化 Serailizable

### 数据库中间件  

Mycat





### 1. 能说下myisam 和 innodb的区别吗？

myisam引擎是5.1版本之前的默认引擎，支持全文检索、压缩、空间函数等，但是不支持事务和行级锁，所以一般用于有大量查询少量插入的场景来使用，而且myisam不支持外键，并且索引和数据是分开存储的。

innodb是基于聚簇索引建立的，和myisam相反它支持事务、外键，并且通过MVCC来支持高并发，索引和数据存储在一起。

### 2. 说下mysql的索引有哪些吧，聚簇和非聚簇索引又是什么？

索引按照数据结构来说主要包含B+树和Hash索引。

假设我们有张表，结构如下：

```sql
create table user(
 id int(11) not null,
  age int(11) not null,
  primary key(id),
  key(age)
);
```

B+树是左小右大的顺序存储结构，节点只包含id索引列，而叶子节点包含索引列和数据，这种数据和索引在一起存储的索引方式叫做聚簇索引，一张表只能有一个聚簇索引。假设没有定义主键，InnoDB会选择一个唯一的非空索引代替，如果没有的话则会隐式定义一个主键作为聚簇索引。

![图片](../../images/640-20210604154637597)

这是主键聚簇索引存储的结构，那么非聚簇索引的结构是什么样子呢？非聚簇索引(二级索引)保存的是主键id值，这一点和myisam保存的是数据地址是不同的。

![图片](../../images/640-20210604154745888)

最终，我们一张图看看InnoDB和Myisam聚簇和非聚簇索引的区别

![图片](../../images/640-20210604154307792)

### 3. 那你知道什么是覆盖索引和回表吗？

覆盖索引指的是在一次查询中，如果一个索引包含或者说覆盖所有需要查询的字段的值，我们就称之为覆盖索引，而不再需要回表查询。

而要确定一个查询是否是覆盖索引，我们只需要explain sql语句看Extra的结果是否是“Using index”即可。

以上面的user表来举例，我们再增加一个name字段，然后做一些查询试试。

```sql
explain select * from user where age=1; //查询的name无法从索引数据获取
explain select id,age from user where age=1; //可以直接从索引获取
```

### 4. 锁的类型有哪些呢

mysql锁分为**共享锁**和**排他锁**，也叫做读锁和写锁。

读锁是共享的，可以通过lock in share mode实现，这时候只能读不能写。

写锁是排他的，它会阻塞其他的写锁和读锁。从颗粒度来区分，可以分为**表锁**和**行锁**两种。

表锁会锁定整张表并且阻塞其他用户对该表的所有读写操作，比如alter修改表结构的时候会锁表。

行锁又可以分为**乐观锁**和**悲观锁**，悲观锁可以通过for update实现，乐观锁则通过版本号实现。

### 5. 你能说下事务的基本特性和隔离级别吗？

事务基本特性ACID分别是：

**原子性**指的是一个事务中的操作要么全部成功，要么全部失败。

**一致性**指的是数据库总是从一个一致性的状态转换到另外一个一致性的状态。比如A转账给B100块钱，假设中间sql执行过程中系统崩溃A也不会损失100块，因为事务没有提交，修改也就不会保存到数据库。

**隔离性**指的是一个事务的修改在最终提交前，对其他事务是不可见的。

**持久性**指的是一旦事务提交，所做的修改就会永久保存到数据库中。

而隔离性有4个隔离级别，分别是：

**read uncommit** 读未提交，可能会读到其他事务未提交的数据，也叫做脏读。

用户本来应该读取到id=1的用户age应该是10，结果读取到了其他事务还没有提交的事务，结果读取结果age=20，这就是脏读。

![图片](../../images/640-20210604154307809)

**read commit** 读已提交，两次读取结果不一致，叫做不可重复读。

不可重复读解决了脏读的问题，他只会读取已经提交的事务。

用户开启事务读取id=1用户，查询到age=10，再次读取发现结果=20，在同一个事务里同一个查询读取到不同的结果叫做不可重复读。

![img](../../images/640-20210604154558358)

**repeatable read** 可重复复读，这是mysql的默认级别，就是每次读取结果都一样，但是有可能产生幻读。

**serializable** 串行，一般是不会使用的，他会给每一行读取的数据加锁，会导致大量超时和锁竞争的问题。

### 6. 那ACID靠什么保证的呢？

A原子性由undo log日志保证，它记录了需要回滚的日志信息，事务回滚时撤销已经执行成功的sql

C一致性一般由代码层面来保证

I隔离性由MVCC来保证

D持久性由内存+redo log来保证，mysql修改数据同时在内存和redo log记录这次操作，事务提交的时候通过redo log刷盘，宕机的时候可以从redo log恢复

### 7. 那你说说什么是幻读，什么是MVCC？

要说幻读，首先要了解MVCC，MVCC叫做多版本并发控制，实际上就是保存了数据在某个时间节点的快照。

我们每行数实际上隐藏了两列，创建时间版本号，过期(删除)时间版本号，每开始一个新的事务，版本号都会自动递增。

还是拿上面的user表举例子，假设我们插入两条数据，他们实际上应该长这样。

| id   | name | create_version | delete_version |
| ---- | ---- | -------------- | -------------- |
| 1    | 张三 | 1              |                |
| 2    | 李四 | 2              |                |

这时候假设小明去执行查询，此时current_version=3

```sql
select * from user where id<=3;
```

同时，小红在这时候开启事务去修改id=1的记录，current_version=4

```sql
update user set name='张三三' where id=1;
```

执行成功后的结果是这样的

| id   | name   | create_version | delete_version |
| ---- | ------ | -------------- | -------------- |
| 1    | 张三   | 1              |                |
| 2    | 李四   | 2              |                |
| 1    | 张三三 | 4              |                |

如果这时候还有小黑在删除id=2的数据，current_version=5，执行后结果是这样的。

| id   | name   | create_version | delete_version |
| ---- | ------ | -------------- | -------------- |
| 1    | 张三   | 1              |                |
| 2    | 李四   | 2              | 5              |
| 1    | 张三三 | 4              |                |

由于MVCC的原理是查找创建版本小于或等于当前事务版本，删除版本为空或者大于当前事务版本，小明的真实的查询应该是这样

```sql
select * from user where id<=3 and create_version<=3 and (delete_version>3 or delete_version is null);
```

所以小明最后查询到的id=1的名字还是'张三'，并且id=2的记录也能查询到。这样做是**为了保证事务读取的数据是在事务开始前就已经存在的，要么是事务自己插入或者修改的**。

明白MVCC原理，我们来说什么是幻读就简单多了。举一个常见的场景，用户注册时，我们先查询用户名是否存在，不存在就插入，假定用户名是唯一索引。

1. 小明开启事务current_version=6查询名字为'王五'的记录，发现不存在。
2. 小红开启事务current_version=7插入一条数据，结果是这样：

| id   | Name | create_version | delete_version |
| ---- | ---- | -------------- | -------------- |
| 1    | 张三 | 1              |                |
| 2    | 李四 | 2              |                |
| 3    | 王五 | 7              |                |

1. 小明执行插入名字'王五'的记录，发现唯一索引冲突，无法插入，这就是幻读。

### 8. 那你知道什么是间隙锁吗？

间隙锁是可重复读级别下才会有的锁，结合MVCC和间隙锁可以解决幻读的问题。我们还是以user举例，假设现在user表有几条记录

| id   | Age  |
| ---- | ---- |
| 1    | 10   |
| 2    | 20   |
| 3    | 30   |

当我们执行：

```sql
begin;
select * from user where age=20 for update;

begin;
insert into user(age) values(10); #成功
insert into user(age) values(11); #失败
insert into user(age) values(20); #失败
insert into user(age) values(21); #失败
insert into user(age) values(30); #失败
```

只有10可以插入成功，那么因为表的间隙mysql自动帮我们生成了区间(左开右闭)

```
(negative infinity，10],(10,20],(20,30],(30,positive infinity)
```

由于20存在记录，所以(10,20]，(20,30]区间都被锁定了无法插入、删除。

如果查询21呢？就会根据21定位到(20,30)的区间(都是开区间)。

需要注意的是唯一索引是不会有间隙索引的。

### 9. 你们数据量级多大？分库分表怎么做的？

首先分库分表分为垂直和水平两个方式，一般来说我们拆分的顺序是先垂直后水平。

**垂直分库**

基于现在微服务拆分来说，都是已经做到了垂直分库了

![图片](../../images/640-20210604155130779)

**垂直分表**

如果表字段比较多，将不常用的、数据较大的等等做拆分

![图片](../../images/640-20210604154307808)

**水平分表**

首先根据业务场景来决定使用什么字段作为分表字段(sharding_key)，比如我们现在日订单1000万，我们大部分的场景来源于C端，我们可以用user_id作为sharding_key，数据查询支持到最近3个月的订单，超过3个月的做归档处理，那么3个月的数据量就是9亿，可以分1024张表，那么每张表的数据大概就在100万左右。

比如用户id为100，那我们都经过hash(100)，然后对1024取模，就可以落到对应的表上了。

### 10. 那分表后的ID怎么保证唯一性的呢？

因为我们主键默认都是自增的，那么分表之后的主键在不同表就肯定会有冲突了。有几个办法考虑：

1. 设定步长，比如1-1024张表我们分别设定1-1024的基础步长，这样主键落到不同的表就不会冲突了。
2. 分布式ID，自己实现一套分布式ID生成算法或者使用开源的比如雪花算法这种
3. 分表后不使用主键作为查询依据，而是每张表单独新增一个字段作为唯一主键使用，比如订单表订单号是唯一的，不管最终落在哪张表都基于订单号作为查询依据，更新也一样。

### 11. 分表后非sharding_key的查询怎么处理呢？

1. 可以做一个mapping表，比如这时候商家要查询订单列表怎么办呢？不带user_id查询的话你总不能扫全表吧？所以我们可以做一个映射关系表，保存商家和用户的关系，查询的时候先通过商家查询到用户列表，再通过user_id去查询。
2. 打宽表，一般而言，商户端对数据实时性要求并不是很高，比如查询订单列表，可以把订单表同步到离线（实时）数仓，再基于数仓去做成一张宽表，再基于其他如es提供查询服务。
3. 数据量不是很大的话，比如后台的一些查询之类的，也可以通过多线程扫表，然后再聚合结果的方式来做。或者异步的形式也是可以的。

```java
List<Callable<List<User>>> taskList = Lists.newArrayList();
for (int shardingIndex = 0; shardingIndex < 1024; shardingIndex++) {
    taskList.add(() -> (userMapper.getProcessingAccountList(shardingIndex)));
}
List<ThirdAccountInfo> list = null;
try {
    list = taskExecutor.executeTask(taskList);
} catch (Exception e) {
    //do something
}

public class TaskExecutor {
    public <T> List<T> executeTask(Collection<? extends Callable<T>> tasks) throws Exception {
        List<T> result = Lists.newArrayList();
        List<Future<T>> futures = ExecutorUtil.invokeAll(tasks);
        for (Future<T> future : futures) {
            result.add(future.get());
        }
        return result;
    }
}
```

### 12. 说说mysql主从同步怎么做的吧？

首先先了解mysql主从同步的原理

1. master提交完事务后，写入binlog
2. slave连接到master，获取binlog
3. master创建dump线程，推送binglog到slave
4. slave启动一个IO线程读取同步过来的master的binlog，记录到relay log中继日志中
5. slave再开启一个sql线程读取relay log事件并在slave执行，完成同步
6. slave记录自己的binglog

![图片](../../images/640-20210604154307811)

由于mysql默认的复制方式是异步的，主库把日志发送给从库后不关心从库是否已经处理，这样会产生一个问题就是假设主库挂了，从库处理失败了，这时候从库升为主库后，日志就丢失了。由此产生两个概念。

**全同步复制**

主库写入binlog后强制同步日志到从库，所有的从库都执行完成后才返回给客户端，但是很显然这个方式的话性能会受到严重影响。

**半同步复制**

和全同步不同的是，半同步复制的逻辑是这样，从库写入日志成功后返回ACK确认给主库，主库收到至少一个从库的确认就认为写操作完成。

### 13. 那主从的延迟怎么解决呢？

这个问题貌似真的是个无解的问题，只能是说自己来判断了，需要走主库的强制走主库查询。







# mybatis优缺点？

### 1.mybatis一级缓存与二级缓存的区别？

### 2.mybatis的工作原理？

### 3.Mapper方法可以重载吗？



























# 设计模式

### 单例设计模式（懒汉，饿汉）

```java
public class Hungry {		// 缺点：类加载时就初始化，浪费内存
  private static Hungry instance = new Hungry();
  public static Hungry getInstance() {
   	return instance; 
  }
}

public class Lazy {
  private static void Lazy instance;
  public static Lazy getInstance() {
    if (instance == null) {
      synchronized(Lazy.class) {
        //假设没有第二层检查，那么第一个线程创建完对象释放锁后，后面进入对象也会创建对象，会产生多个对象
        if (instance == null) {
          //volatile关键字作用为禁止指令重排，
          instance = new Lazy();
          //singleton=new Singleton语句为非原子性，实际上会执行以下内容：
          //(1)在堆上开辟空间；(2)属性初始化;(3)引用指向对象
        }
      }
    }
    return instance;
  }
}
```































# 前后端分离

在前后端彻底分离这一时期，前端的范围被扩展，controller层也被认为属于前端的一部分。在这一时期：

- 前端：负责View和Controller层。
- 后端：只负责Model层，业务/数据处理等。

可是服务端人员对前端HTML结构不熟悉，前端也不懂后台代码呀，controller层如何实现呢？这就是node.js的妙用了，node.js适合运用在高并发、I/O密集、少量业务逻辑的场景。最重要的一点是，前端不用再学一门其他的语言了，对前端来说，上手度大大提高。

![图片](../../images/640-20210406105355994.png)

可以就把Nodejs当成跟前端交互的api。总得来说，Nodejs的作用在mvc中相当于C（控制器）。Nodejs路由的实现逻辑是把前端静态页面代码当成字符串发送到客户端（例如浏览器），简单理解可以理解为路由是提供给客户端的一组api接口，只不过返回的数据是页面代码的字符串而已。

用NodeJs来作为桥梁架接服务器端API输出的JSON。后端出于性能和别的原因，提供的接口所返回的数据格式也许不太适合前端直接使用，前端所需的排序功能、筛选功能，以及到了视图层的页面展现，也许都需要对接口所提供的数据进行二次处理。这些处理虽可以放在前端来进行，但也许数据量一大便会浪费浏览器性能。因而现今，增加Node中间层便是一种良好的解决方案。

**增加node.js作为中间层，具体有哪些好处呢？**

**1、适配性提升:** 我们其实在开发过程中，经常会给PC端、mobile、app端各自研发一套前端。其实对于这三端来说，大部分端业务逻辑是一样的。唯一区别就是交互展现逻辑不同。

如果controller层在后端手里，后端为了这些不同端页面展示逻辑，自己维护这些controller，模版无法重用，徒增和前端沟通端成本。如果增加了node.js层，此时架构图如下：

![图片](../../images/640-20210406110158078)

在该结构下，每种前端的界面展示逻辑由node层自己维护。如果产品经理中途想要改动界面什么的，可以由前端自己专职维护，后端无需操心。前后端各司其职，后端专注自己的业务逻辑开发，前端专注产品效果开发。

**2、响应速度提升；** 我们有时候，会遇到后端返回给前端的数据太简单了，前端需要对这些数据进行逻辑运算。那么在数据量比较小的时候，对其做运算分组等操作，并无影响。但是当数据量大的时候，会有明显的卡顿效果。这时候，node中间层其实可以将很多这样的代码放入node层处理、也可以替后端分担一些简单的逻辑、又可以用模板引擎自己掌握前台的输出。这样做灵活度、响应度都大大提升。

**3、性能得到提升；** 大家应该都知道单一职责原则。从该角度来看，我们，请求一个页面，可能要响应很多个后端接口，请求变多了，自然速度就变慢了，这种现象在mobile端更加严重。采用node作为中间层，将页面所需要的多个后端数据，直接在内网阶段就拼装好，再统一返回给前端，会得到更好的性能。

**4、异步与模板统一；** 淘宝首页就是被几十个HTML片段（每个片段一个文件）拼装成，之前PHP同步include这几十个片段，一定是串行的，Node可以异步，读文件可以并行，一旦这些片段中也包含业务逻辑，异步的优势就很明显了，真正做到哪个文件先渲染完就先输出显示。

前端机的文件系统越复杂，页面的组成片段越多，这种异步的提速效果就越明显。前后端模板统一在无线领域很有用，PC页面和WIFI场景下的页面适合前端渲染（后端数据Ajax到前端），2G、3G弱网络环境适合后端渲染（数据随页面吐给前端），所以同样的模板，在不同的条件下走不同的渲染渠道，模板只需一次开发。

增加NodeJS中间层后的前后端职责划分：

![图片](../../images/640-20210406110230489)