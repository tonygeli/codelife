[TOC]











### CompletableFuture

这个completableFuture是JDK1.8版本新引入的类。下面是这个类。实现了俩接口。本身是个class。这个是Future的实现类。

使用completionStage接口去支持完成时触发的函数和操作。

一个completetableFuture就代表了一个任务。他能用Future的方法。还能做一些之前说的executorService配合futures做不了的。

之前future需要等待isDone为true才能知道任务跑完了。或者就是用get方法调用的时候会出现阻塞。而使用completableFuture的使用就可以用then，when等等操作来防止以上的阻塞和轮询isDone的现象出现。



初始类加载器

​	触发符号引用-类

定义类

​	双亲委派原则

类名+定义类加载器完全一样才是同一个类



Sep





```
public static void main() {
	Count
}
```



## 线上开启xdebug

```shell
SCF_BASE_PKG="-Dscf.serializer.basepakage=com.anjuke;com.bj58"
if [[ ${WCloud_Env} == 'Product' || ${WCloud_Env} == 'SandBox' ]];then
    echo "switch online"
    cp online/*  config/
    JAVA_OPTS="${JAVA_OPTS} -Dspring.profiles.active=online"
    test ${WCloud_Env} == 'SandBox' && JAVA_OPTS="${JAVA_OPTS} -DenabledMock=true -Xdebug -Xrunjdwp:transport=dt_socket,address=9000,server=y,suspend=n "
else
    echo "switch test"
    cp offline/*  config/
    JAVA_OPTS="${JAVA_OPTS_TEST} -Dspring.profiles.active=offline -DenabledMock=true -Xdebug -Xrunjdwp:transport=dt_socket,address=9000,server=y,suspend=n "
fi
```



# Spring



## 1.容器的启动

```java
BeanFactory
	ApplicationContext
```





# JVM

线程私有的：程序计数器，虚拟机栈，本地方法栈
线程共享的：堆，方法区，直接内存

1.**程序计数器**：是唯一不会出现 OutOfMemoryError 的内存区域，它的生命周期随着线程的创建而创建，随着线程的结束而死亡。

2.**虚拟机栈**：虚拟机栈是由一个个栈帧组成，线程在执行一个方法时，便会向栈中放入一个栈帧，每个栈帧中都拥有局部变量表、操作数栈、动态链接、方法出口信息。局部变量表存放基本数据类型（boolean、byte、char、short、int、float、long、double）和对象引用（reference)。
虚拟机栈会出现两种异常：StackOverFlowError 和 OutOfMemoryError。
	StackOverFlowError：若Java虚拟机栈的内存大小不允许动态扩展，那么当线程请求栈的深度超过当前Java虚拟机栈的最大深度的时候，就抛出StackOverFlowError异常。
	OutOfMemoryError：若 Java 虚拟机栈的内存大小允许动态扩展，且当线程请求栈时内存用完了，无法再动态扩展了，此时抛出OutOfMemoryError异常。

4.堆是Java 虚拟机所管理的内存中最大的一块，几乎所有的对象实例以及数组都在这里分配内存。也被称作GC堆，收集器基本都采用分代垃圾收集算法。“分代回收”是基于这样一个事实：对象的生命周期不同，所以针对不同生命周期的对象可以采取不同的回收方式，以便提高回收效率。



读写锁 ReadWriteLock

stampedLock  乐观读





## 如何相互转换逗号分隔的字符串和List

### 一、**将逗号分隔的字符串转换为List**

方法 1： 利用JDK的Arrays类

```java
String str = "a,b,c";
List<String> result = Arrays.asList(str.split(","));
```

方法 2： 利用Guava的Splitter

```java
String str = "a, b, c";
List<String> result = Splitter.on(",").trimResults().splitToList(str);
```

方法 3： 利用Apache Commons的StringUtils （只是用了split)

```java
String str = "a,b,c";
List<String> result = Arrays.asList(StringUtils.split(str,","));
```

方法 4: 利用Spring Framework的StringUtils

```java
String str = "a,b,c";

List<String> str = Arrays.asList(StringUtils.commaDelimitedListToStringArray(str));
```



### 二、将List转换为逗号分隔符

方法 1： 利用JDK  (好像没有很好的方法，需要一步一步实现）

方法 2： 利用Guava的Joiner

```java
List<String> list = Arrays.asList("a", "b");
String str = Joiner.on(",").join(list);
```

方法 3： 利用Apache Commons的StringUtils

```java
List<String> list = Arrays.asList("a", "b");
String str = StringUtils.join(list.toArray(), ",");
```

方法 4：利用Spring Framework的StringUtils

```java
List<String> list = Arrays.asList("a", "b");
String str = StringUtils.collectionToDelimitedString(list, ",");
```









备忘录





























