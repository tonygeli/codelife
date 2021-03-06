# TCP粘包、半包问题和Netty编解码框架

## 协议栈功能设计

- 基于Netty的NIO通信框架，提供高性能的异步通信能力
- 提供消息的编解码框架，可以实现POJO的序列化和反序列化
- 提供基于IP地址的白名单介入认证机制
- 链路的有效性校验机制
- 链路的断连重连机制





网络协议和Netty常见面试题汇总

1. TCP的三次握手过程
2. TCP握手为什么需要三次
3. TCP的四次挥手
4. TCP四次挥手要有TIME_WAIT状态？
5. 概述下什么是DDOS攻击和SYN洪水攻击
   1. DDOS使服务器无法正常处理请求，大量肉机
   2. SYN利用三次握手，不对服务器的应答反馈导致服务器一直等待
6. 那些应用比较适用UDP实现？
   1. DNS
7. HTTP和HTTPS的区别
   1. SSL证书 加密
   2. 公钥加密传输、私钥解密
   3. 端口不一样443
8. Netty的特点？为什么使用Netty？
   1. 高性能网络IO
   2. Epoll功能群的Bug
   3. volatile  多线程的安全与性能
   4. 引入内存池
9. 概述下Netty的线程模型
10. Netty中有哪些重要组件
11. TCP粘包、拆包的原因及解决方法？
12. 介绍下序列化
13. Netty是如何解决JDK中的Selector Bug？
    1. Linux内核
14. Netty高性能表现在哪些方面？
15. Netty发送消息有几种方式？
16. Netty的内存管理机制是什么？
17. ByteBuf的特点



