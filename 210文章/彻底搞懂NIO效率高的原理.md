# 彻底搞懂NIO效率高的原理

**前言**

这篇文章读不懂的没关系，可以先收藏一下。笔者准备介绍完epoll和NIO等知识点，然后写一篇Java网络IO模型的介绍，这样可以使Java网络IO的知识体系更加地完整和严谨。初学者也可以等看完IO模型介绍的博客之后，再回头看这些博客，会更加有收获。

# **NIO相比BIO的优势**

NIO（Non-blocking I/O，在Java领域，也称为New I/O），是一种同步非阻塞的I/O模型，也是I/O多路复用的基础，已经被越来越多地应用到大型应用服务器，成为解决高并发与大量连接、I/O处理问题的有效方式。

![彻底搞懂NIO效率高的原理](../../images/f694de00677c45a1abf7d87aed6cc82a)



bio与nio

**面向流与面向缓冲**

Java NIO和BIO之间第一个最大的区别是，BIO是面向流的，NIO是面向缓冲区的。 JavaIO面向流意味着每次从流中读一个或多个字节，直至读取所有字节，它们没有被缓存在任何地方。此外，它不能前后移动流中的数据。如果需要前后移动从流中读取的数据，需要先将它缓存到一个缓冲区。Java NIO的缓冲读取方法略有不同。数据读取到一个缓冲区，需要时可在缓冲区中前后移动。这就增加了处理过程中的灵活性。但是，还需要检查是否该缓冲区中包含所有需要处理的数据。而且，需确保当更多的数据读入缓冲区时，不要覆盖缓冲区里尚未处理的数据。

> 有关面向缓冲读取数据的示例和注意点，可以点击查看[TCP粘拆包详解与Netty代码示例](https://www.toutiao.com/i6715713088542212622/?group_id=6715713088542212622)

**阻塞IO与非阻塞IO**

Java IO的各种流是阻塞的。这意味着，当一个线程调用read() 或write()时，该线程被阻塞，直到有数据被读取或者数据写入。该线程在阻塞期间不能做其他事情。而Java NIO的非阻塞模式，如果通道没有东西可读，或不可写，读写函数马上返回，而不会阻塞，这个线程可以去做别的事情。 线程通常将非阻塞IO的空闲时间用于在其它通道上执行IO操作，所以一个单独的线程可以管理多个输入和输出通道（channel），即IO多路复用的原理。

**零拷贝**

在传统的文件IO操作中，我们都是调用操作系统提供的底层标准IO系统调用函数read()、write() ，此时调用此函数的进程（在JAVA中即java进程）由当前的用户态切换到内核态，然后OS的内核代码负责将相应的文件数据读取到内核的IO缓冲区，然后再把数据从内核IO缓冲区拷贝到进程的私有地址空间中去，这样便完成了一次IO操作。

![彻底搞懂NIO效率高的原理](../../images/81a8c1b3ec94496db62c4460789e1d37)



IO

而NIO的零拷贝与传统的文件IO操作最大的不同之处就在于它虽然也是要从磁盘读取数据，但是它并不需要将数据读取到OS内核缓冲区，而是直接将进程的用户私有地址空间中的一部分区域与文件对象建立起映射关系，这样直接从内存中读写文件，速度大幅度提升。

![彻底搞懂NIO效率高的原理](../../images/5d63b9ca6ae44b93b8431d093ef8b615)



NIO

> 详细的解析，之后会有单独的博客进行讲解

# **NIO的核心部分**

Java NIO主要由以下三个核心部分组成：

- Channel
- Buffer
- Selector

**Channel**

基本上，所有的IO在NIO中都从一个Channel开始。数据可以从Channel读到Buffer中，也可以从Buffer写到Channel中。这里有个图示：

![彻底搞懂NIO效率高的原理](../../images/a1097d0cf9c6497389c7c3ce1e34d6d2)



channel与buffer

Channel和Buffer有好几种类型。下面是Java NIO中的一些主要Channel的实现：

- FileChannel(file)
- DatagramChannel(UDP)
- SocketChannel(TCP)
- ServerSocketChannel(TCP)

这些通道涵盖了UDP和TCP网络IO以及文件IO。

> 最后两个channel的关系。通过
> ServerSocketChannel.accept() 方法监听新进来的连接。当 accept()方法返回的时候,它返回一个包含新进来的连接的 SocketChannel。因此, accept()方法会一直阻塞到有新连接到达。通常不会仅仅只监听一个连接,在while循环中调用 accept()方法.

//打开 ServerSocketChannel

ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

serverSocketChannel.socket().bind(new InetSocketAddress(9999));

while(true){

SocketChannel socketChannel = serverSocketChannel.accept();

//do something with socketChannel...

}

//关闭ServerSocketChannel

serverSocketChannel.close();

**Buffer**

缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成NIO Buffer对象，并提供了一组方法，用来方便的访问该块内存。

Java NIO里关键的Buffer实现：

- ByteBuffer
- CharBuffer
- DoubleBuffer
- FloatBuffer
- IntBuffer
- LongBuffer
- ShortBuffer

这些Buffer覆盖了你能通过IO发送的基本数据类型：byte、short、int、long、float、double和char。

为了理解Buffer的工作原理，需要熟悉它的三个属性：

- capacity
- position
- limit

position和limit的含义取决于Buffer处在读模式还是写模式。不管Buffer处在什么模式，capacity的含义总是一样的。

![彻底搞懂NIO效率高的原理](../../images/0f8892655d5141ed93e9a4442670f30b)



buffer模型

**capacity**

作为一个内存块，Buffer有个固定的最大值，就是capacity。Buffer只能写capacity个byte、long、char等类型。一旦Buffer满了，需要将其清空（通过读数据或者清除数据）才能继续写数据往里写数据。

**position**

当写数据到Buffer中时，position表示当前的位置。初始的position值为0。当一个byte、long等数据写到Buffer后， position会向前移动到下一个可插入数据的Buffer单元。position最大可为capacity – 1.

当读取数据时，也是从某个特定位置读。当将Buffer从写模式切换到读模式，position会被重置为0。 当从Buffer的position处读取数据时，position向前移动到下一个可读的位置。

**limit**

在写模式下，Buffer的limit表示最多能往Buffer里写多少数据。 写模式下，limit等于capacity。

当切换Buffer到读模式时， limit表示你最多能读到多少数据。因此，当切换Buffer到读模式时，limit会被设置成写模式下的position值。

**Selector**

Selector允许单线程处理多个 Channel。如果你的应用打开了多个连接（通道），但每个连接的流量都很低，使用Selector就会很方便。例如，在一个聊天服务器中。

这是在一个单线程中使用一个Selector处理3个Channel的图示：

![彻底搞懂NIO效率高的原理](../../images/12d44b7ddffd4908bfe9627f63580ec1)



Selector

要使用Selector，得向Selector注册Channel，然后调用它的select()方法。这个方法会一直阻塞到某个注册的通道有事件就绪。一旦这个方法返回，线程就可以处理这些事件，事件例如有新连接进来，数据接收等。

**NIO与epoll的关系**

Java NIO根据操作系统不同， 针对NIO中的Selector有不同的实现：

- macosx:KQueueSelectorProvider
- solaris:DevPollSelectorProvider
- Linux:EPollSelectorProvider (Linux kernels >= 2.6)或PollSelectorProvider
- windows:WindowsSelectorProvider

所以不需要特别指定，Oracle JDK会自动选择合适的Selector。

如果想设置特定的Selector，可以设置属性，例如：

-Djava.nio.channels.spi.SelectorProvider=sun.nio.ch.EPollSelectorProvider

JDK在Linux已经默认使用epoll方式，但是JDK的epoll采用的是水平触发，所以Netty自4.0.16起, Netty为Linux通过JNI的方式提供了native socket transport。Netty重新实现了epoll机制，

1. 采用边缘触发方式
2. netty epoll transport暴露了更多的nio没有的配置参数，如 TCP_CORK, SO_REUSEADDR等等。
3. C代码，更少GC，更少synchronized

使用native socket transport的方法很简单，只需将相应的类替换即可。

```
NioEventLoopGroup → EpollEventLoopGroup
NioEventLoop → EpollEventLoop
NioServerSocketChannel → EpollServerSocketChannel
NioSocketChannel → EpollSocketChannel 
```

有关epoll的详细讲解，可以点击查看[彻底搞懂epoll高效运行的原理](https://www.toutiao.com/i6720907162404520456/?group_id=6720907162404520456)

# **NIO处理消息的核心思路**

结合示例代码，总结NIO的核心思路：

1. NIO 模型中通常会有两个线程，每个线程绑定一个轮询器 selector ，在上面例子中serverSelector负责轮询是否有新的连接，clientSelector负责轮询连接是否有数据可读
2. 服务端监测到新的连接之后，不再创建一个新的线程，而是直接将新连接绑定到clientSelector上，这样就不用BIO模型中1w 个while循环在阻塞，参见(1)
3. clientSelector被一个 while 死循环包裹着，如果在某一时刻有多条连接有数据可读，那么通过clientSelector.select(1)方法可以轮询出来，进而批量处理，参见(2)
4. 数据的读写面向 Buffer，参见(3)

**NIO的示例代码**

```
public class NIOServer {
 public static void main(String[] args) throws IOException {
 Selector serverSelector = Selector.open();
 Selector clientSelector = Selector.open();
 new Thread(() -> {
 try {
 // 对应IO编程中服务端启动
 ServerSocketChannel listenerChannel = ServerSocketChannel.open();
 listenerChannel.socket().bind(new InetSocketAddress(8000));
 listenerChannel.configureBlocking(false);
 listenerChannel.register(serverSelector, SelectionKey.OP_ACCEPT);
 while (true) {
 // 监测是否有新的连接，这里的1指的是阻塞的时间为 1ms
 if (serverSelector.select(1) > 0) {
 Set<SelectionKey> set = serverSelector.selectedKeys();
 Iterator<SelectionKey> keyIterator = set.iterator();
 while (keyIterator.hasNext()) {
 SelectionKey key = keyIterator.next();
 if (key.isAcceptable()) {
 try {
 // (1) 每来一个新连接，不需要创建一个线程，而是直接注册到clientSelector
 SocketChannel clientChannel = ((ServerSocketChannel) key.channel()).accept();
 clientChannel.configureBlocking(false);
 clientChannel.register(clientSelector, SelectionKey.OP_READ);
 } finally {
 keyIterator.remove();
 }
 }
 }
 }
 }
 } catch (IOException ignored) {
 }
 }).start();
 new Thread(() -> {
 try {
 while (true) {
 // (2) 批量轮询是否有哪些连接有数据可读，这里的1指的是阻塞的时间为 1ms
 if (clientSelector.select(1) > 0) {
 Set<SelectionKey> set = clientSelector.selectedKeys();
 Iterator<SelectionKey> keyIterator = set.iterator();
 while (keyIterator.hasNext()) {
 SelectionKey key = keyIterator.next();
 if (key.isReadable()) {
 try {
 SocketChannel clientChannel = (SocketChannel) key.channel();
 ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
 // (3) 面向 Buffer
 clientChannel.read(byteBuffer);
 byteBuffer.flip();
 System.out.println(Charset.defaultCharset().newDecoder().decode(byteBuffer)
 .toString());
 } finally {
 keyIterator.remove();
 key.interestOps(SelectionKey.OP_READ);
 }
 }
 }
 }
 }
 } catch (IOException ignored) {
 }
 }).start();
 }
}
```

更多内容，欢迎关注微信公众号：全菜工程师小辉~