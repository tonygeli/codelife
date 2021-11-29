# TCP粘拆包详解与Netty代码示例

TCP是个“流”协议，所谓流，就是没有界限的一串数据。可以想想河里的流水，是连成一片的，其间并没有分界线。TCP底层并不了解上层业务数据的具体含义，它会根据TCP缓冲区的实际情况进行包的划分，所以在业务上认为，一个完整的包可能会被TCP拆分成多个包进行发送，也有可能把多个小的包封装成一个大的数据包发送，这就是所谓的TCP粘包和拆包问题。

> 有关TCP的详细讲解，可以点击[关于三次握手与四次挥手你要知道这些](https://www.toutiao.com/i6711623920568500743/?group_id=6711623920568500743)和[快速了解TCP的流量控制与拥塞控制](https://www.toutiao.com/i6711818943989809671/?group_id=6711818943989809671)

**TCP粘包或拆包的原因**

1. 应用程序写入的数据大于套接字缓冲区大小，这将会发生拆包。
2. 应用程序写入数据小于套接字缓冲区大小，网卡将应用多次写入的数据发送到网络上，这将会发生粘包。
3. 进行MSS（最大报文长度）大小的TCP分段，当TCP报文长度-TCP头部长度>MSS的时候将发生拆包。
4. 接收方法不及时读取套接字缓冲区数据，这将发生粘包。

**拆包和粘包的形式**

第一种情况：接收端正常收到两个数据包，即没有发生拆包和粘包的现象，此种情况不在本文的讨论范围内。

![TCP粘拆包详解与Netty代码示例](../../images/bb8b60d1c9ae493dbe3557c0017ffa15.png)



发生拆包

第二种情况：接收端只收到一个数据包，由于TCP是不会出现丢包的，所以这一个数据包中包含了发送端发送的两个数据包的信息，这种现象即为粘包。这种情况由于接收端不知道这两个数据包的界限，所以对于接收端来说很难处理。

![TCP粘拆包详解与Netty代码示例](../../images/fb71bd0fd3ed46d48eae951131744c1e.png)



发生粘包

第三种情况：这种情况有两种表现形式，如下图。接收端收到了两个数据包，但是这两个数据包要么是不完整的，要么就是多出来一块，这种情况即发生了拆包和粘包。这两种情况如果不加特殊处理，对于接收端同样是不好处理的。

![TCP粘拆包详解与Netty代码示例](../../images/5a0a8ba5a89143b688b0c90717c27f49.png)



发生拆包和粘包

![TCP粘拆包详解与Netty代码示例](../../images/8bbe54ff2fbd431e8fc9568c9bfad325.png)



发生拆包和粘包

**粘包和拆包的解决办法**

1. 发送端给每个数据包添加包首部，首部中应该至少包含数据包的长度，这样接收端在接收到数据后，通过读取包首部的长度字段，便知道每一个数据包的实际长度了。
2. 发送端将每个数据包封装为固定长度（不够的可以通过补0填充），这样接收端每次从接收缓冲区中读取固定长度的数据就自然而然的把每个数据包拆分开来。
3. 可以在数据包之间设置边界，添加特殊符号（如：回车符），这样，接收端通过这个边界就可以将不同的数据包拆分开。

**Netty中的代码示例**

Netty封装了JDK的NIO，是一个异步事件驱动的网络应用框架，用于快速开发可维护的高性能服务器和客户端。一般开发中并不会用JDK原生NIO，原因如下：

1. 使用JDK自带的NIO需要了解太多的概念，编程复杂，一不小心bug横飞
2. Netty底层IO模型随意切换，而这一切只需要做微小的改动，改改参数，Netty可以直接从NIO模型变身为IO模型
3. Netty自带的拆包解包，异常检测等机制让你从NIO的繁重细节中脱离出来，让你只需要关心业务逻辑
4. Netty解决了JDK的很多包括空轮询在内的bug
5. Netty底层对线程，selector做了很多细小的优化，精心设计的reactor线程模型做到非常高效的并发处理
6. 自带各种协议栈让你处理任何一种通用协议都几乎不用亲自动手
7. Netty社区活跃，遇到问题随时邮件列表或者issue
8. Netty已经历各大rpc框架，消息中间件，分布式通信中间件线上的广泛验证，健壮性无比强大

所以，本文选择演示Netty的编解码代码。

在Netty中，我们定义MessageToByteEncoder<T>的继承类，重写其encode函数，来自定义编码器。

```java
public class SocketEncoder extends MessageToByteEncoder<Packet> {
 @Override
 protected void encode(ChannelHandlerContext channelHandlerContext, NetPacket msg, ByteBuf byteBuf) throws Exception {
 		byte body[] = msg.getBody();
		int packetLen = body.length;
 		// 先设置包长度，然后写入二进制数据
 		byteBuf.writeInt(packetLen);
 		byteBuf.writeBytes(body);
 }
}
```

在Netty中，我们定义ByteToMessageDecoder的继承类，重写其decode函数，用来自定义解码器。

```java
public class SocketDecoder extends ByteToMessageDecoder {
 @Override
 void decode(ChannelHandlerContext channelHandlerContext, ByteBuf byteBuf, List<Object> list) throws Exception {
 		int bufLen = byteBuf.readableBytes();
 		// 解决粘包问题（不够一个包头的长度）
 		// 4字节是报文中使用了一个int表示了报文长度
 		if (bufLen < 4) {
			 return;
		}
 		// 标记一下当前的readIndex的位置
 		byteBuf.markReaderIndex();
 		int packetLength = byteBuf.readInt();
 		// 读到的消息体长度如果小于我们传送过来的消息长度，则resetReaderIndex。重置读索引,继续接收
 		if (byteBuf.readableBytes() < packetLength) {
 			// 配合markReaderIndex使用的。把readIndex重置到mark的地方
 			byteBuf.resetReaderIndex();
 			return;
 		}
 		NetPacket netPacket = new NetPacket();
 		netPacket.setPacketLen(packetLength);
 		// 传送过来数据的长度，满足我们的要求了
 		byte body[] = new byte[packetLength];
 		byteBuf.readBytes(body);
 		netPacket.setBody(body);
 		list.add(netPacket);
 }
}
```