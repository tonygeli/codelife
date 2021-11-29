[TOC]

# Object对象简介

**除了**八大基本数据类型。当然了，八大基本数据类型也能**装箱**成为对象



## 1.equals和hashCode方法

重写`equals()`方法，就必须重写`hashCode()`的方法

```java
    public boolean equals(Object anObject) {
        if (this == anObject) {		// 地址相同
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])		// 字符相同
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }

    public int hashCode() {
        int h = hash;
        if (h == 0 && value.length > 0) {
            char val[] = value;

            for (int i = 0; i < value.length; i++) {
                h = 31 * h + val[i];
            }
            hash = h;
        }
        return h;
    }
```



## 2.toString方法

## 3.clone方法

**protected修饰的类和属性,对于自己、本包和其子类可见**

**可能会想**：`clone()`方法是定义在Object类上的(以protected来修饰)，而我们自定义的Address对象**隐式**继承着Object(所有的对象都是Object的子类)，那么子类调用Object以protected来修饰`clone()`是完全没问题的

- 但是，IDE现实告诉我，这**编译就不通过了**！

```java
public class Address  {

    private int provinceNo;
    private int cityNo;
    private int streetNo;

    public Address() {
    }

    public Address(int provinceNo, int cityNo, int streetNo) {
        this.provinceNo = provinceNo;
        this.cityNo = cityNo;
        this.streetNo = streetNo;
    }
}
 Address address = new Address(1, 2, 3);
 address.clone();		// clone() has protected access in 'java.lang.Ojbect'

```

上面的代码就错在：Address与Object**不是在同一个包下**的，而Address直接访问了Object的clone方法。这是不行的。

## 4.wait和notify方法

- 无论是wait、notify还是notifyAll()都需要**由监听器对象(锁对象)来进行调用**

- - 简单来说：**他们都是在同步代码块中调用的**，否则会抛出异常！

- `notify()`唤醒的是在等待队列的**某个**线程(不确定会唤醒哪个)，`notifyAll()`唤醒的是等待队列**所有**线程

- 导致`wait()`的线程被唤醒可以有4种情况

- - 该线程被中断
  - `wait()`时间到了
  - 被`notify()`唤醒
  - 被`notifyAll()`唤醒

- 调用`wait()`的线程会**释放掉锁**

notify方法调用后，被唤醒的线程**不会立马获得到锁对象**。而是等待notify的synchronized代码块**执行完之后**才会获得锁对象

`Thread.sleep()`与`Object.wait()`二者都可以暂停当前线程，释放CPU控制权。

- 主要的区别在于`Object.wait()`在释放CPU同时，**释放了对象锁的控制**。
- 而`Thread.sleep()`没有对锁释放

## 5.finalize()方法

`finalize()`方法将在**垃圾回收器清除对象之前调用**，但该方法不知道何时调用，具有**不定性**

一个对象的finalize()方法**只会被调用一次**，而且finalize()被调用不意味着gc会立即回收该对象，所以有可能调用finalize()后，该对象又不需要被回收了，然后到了真正要被回收的时候，因为前面调用过一次，所以不会调用finalize()，产生问题。