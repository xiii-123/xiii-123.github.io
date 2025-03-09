---
title: Java Note
published: 2025-03-09
description: 'Java学习之路'
image: ''
tags: [java, 学习, 拾遗]
category: 'java'
draft: false 
---
# 拾遗

## 抽象类和接口的区别

在Java编程语言中，抽象类（Abstract Class）和接口（Interface）都是用来定义抽象层次和实现多态的机制。以下是它们之间的一些主要区别：

| 特性     | 抽象类                                         | 接口                                               |
| -------- | ---------------------------------------------- | -------------------------------------------------- |
| 继承     | 一个类只能继承一个抽象类                       | 一个类可以实现多个接口                             |
| 方法实现 | 可以包含具体实现的方法和抽象方法               | 只能包含抽象方法（Java 8+ 默认方法和静态方法除外） |
| 构造器   | 可以有构造器                                   | 不能有构造器                                       |
| 成员变量 | 可以包含非final变量                            | 成员变量默认是public static final的                |
| 方法添加 | 可以添加新的抽象方法或不添加，不对子类构成强制 | 新添加的方法必须被实现类实现（默认方法除外）       |

## 值传递和引用传递

在Java编程语言中，所有的参数传递都是按值传递的，这意味着不管是一个原始数据类型的值还是一个对象的引用，传递的都是一个副本。以下是具体情况的分析：

**值传递（Value Passing）：**

1. **基本数据类型（Primitive Data Types）**：
   - 当你传递一个基本数据类型的变量（如`int`, `float`, `double`, `byte`, `char`, `boolean`等）给一个方法时，实际传递的是数据值的一个副本。在方法内部对这个值的任何操作都不会影响到原始的变量。
2. **字符串（String）**：
   - Java中的`String`对象是不可变的。当你传递一个`String`对象到一个方法时，实际上传递的是对象引用的一个副本。但由于`String`的不可变性，方法内部对字符串的任何修改实际上都会创建一个新的`String`对象，原始的`String`对象保持不变。

**引用传递的误解：**

在Java中，通常所说的“引用传递”实际上是指以下情况：
1. **对象引用**：
   - 当你传递一个对象引用到方法时，你传递的是引用值的副本，这个副本指向同一个对象。因此，如果在方法内部改变了引用指向对象的状态（即改变了对象的字段），那么原始引用所指向的对象也会被改变。
   - 这经常被误解为“引用传递”，但实际上，引用本身是按值传递的，只是引用指向的对象在堆内存中的数据可以被方法改变。
2. **数组**：
   - 数组在Java中是对象，所以当你传递一个数组到方法时，传递的是数组引用的一个副本。方法内部对数组的任何操作都会影响原始数组，因为副本和原始引用指向同一个数组对象。

**特殊情况：**

1. **包装类（Wrapper Classes）**：
   - 基本数据类型的包装类（如`Integer`, `Double`, `Boolean`等）是对象，因此传递它们时也是传递引用的副本。但是，由于它们通常用于封装不可变的数据，所以它们的行为类似于基本数据类型的值传递。
2. **集合类（Collections）**：
   - 集合类（如`List`, `Set`, `Map`等）是对象，因此传递它们时也是传递引用的副本。方法内部对集合内容的修改会影响到原始集合。

总结来说，Java中的参数传递总是按值传递的，但是当传递的是对象引用时，可以间接修改对象的状态，这常常给人一种引用传递的错觉。要准确理解这一点，重要的是要区分引用本身（作为参数传递的值）和引用指向的对象。

## 监控对象数量

在Java中，精确地控制对象的生命周期是非常有限的，因为Java的垃圾收集器（Garbage Collector, GC）会自动管理内存。不过，你可以使用一些设计模式和技术手段来近似实现这一功能。
以下是一些方法：

**使用引用计数**

你可以手动实现引用计数，但这通常不推荐，因为它会增加复杂性，并且Java的垃圾收集器已经足够高效。
```java
public class RefCounter {
    private static final Map<Object, Integer> referenceCounts = new HashMap<>();
    public static void retain(Object obj) {
        referenceCounts.merge(obj, 1, Integer::sum);
    }
    public static void release(Object obj) {
        if (referenceCounts.containsKey(obj)) {
            int count = referenceCounts.get(obj);
            if (count == 1) {
                referenceCounts.remove(obj);
                // 可以在这里调用对象的清理逻辑
            } else {
                referenceCounts.put(obj, count - 1);
            }
        }
    }
    public static int getRefCount(Object obj) {
        return referenceCounts.getOrDefault(obj, 0);
    }
}
```
**使用弱引用和引用队列**

Java提供了弱引用（WeakReference）和引用队列（ReferenceQueue），可以用来监控对象何时变得不可达。
```java
public class WeakReferenceExample {
    private static ReferenceQueue<MyClass> queue = new ReferenceQueue<>();
    private static Map<WeakReference<MyClass>, String> weakReferences = new HashMap<>();
    public static void main(String[] args) {
        MyClass obj = new MyClass();
        WeakReference<MyClass> weakRef = new WeakReference<>(obj, queue);
        weakReferences.put(weakRef, "Object info");
        // 模拟垃圾收集
        obj = null;
        System.gc();
        // 检查对象是否被回收
        Reference<? extends MyClass> ref = queue.poll();
        if (ref != null) {
            weakReferences.remove(ref);
            // 对象已被回收，可以进行清理工作
        }
    }
}
```
**使用try-with-resources或finally块**

对于实现了AutoCloseable接口的资源，可以使用try-with-resources语句来自动管理资源。
```java
public class MyResource implements AutoCloseable {
    public void doSomething() {
        // ...
    }
    @Override
    public void close() throws Exception {
        // 清理资源
    }
}
public class ResourceExample {
    public static void main(String[] args) {
        try (MyResource resource = new MyResource()) {
            resource.doSomething();
        } catch (Exception e) {
            e.printStackTrace();
        }
        // 当try块结束时，resource会自动调用close方法
    }
}
```
**使用显式生命周期管理**

在某些情况下，你可以设计API，要求用户显式地开始和结束对象的生命周期。
```java
public class LifecycleObject {
    private boolean started = false;
    public void start() {
        if (!started) {
            // 开始逻辑
            started = true;
        }
    }
    public void stop() {
        if (started) {
            // 停止逻辑
            started = false;
        }
    }
}
```
使用这些方法，你可以更好地控制对象的生命周期，但要注意，这些方法通常会增加代码的复杂性，并且可能带来性能开销。在设计应用程序时，应该权衡这些因素。

## 访问控制修饰符

### protected

- 基类的 protected 成员是包内可见的，并且对子类可见；
- 若子类与基类不在同一包中，那么在子类中，子类实例可以访问其从基类继承而来的protected方法，而不能访问基类实例的protected方法。

```java
package p1;
class Father {
    protected Object clone() throws CloneNotSupportedException{
       return super.clone();
    }
}
 
package p2;
public class Son extends Father {
    public static void main(String args[]) {
       Father obj = new Father();
       obj.clone(); // Compile Error         ----（1）
 
       Son tobj = new Son();
       tobj.clone(); // Complie OK         ----（2）
    }
}
```

### synchronized

使用`synchronized`关键字修饰的方法在Java中是线程安全的。当某个方法被声明为`synchronized`时，它的锁是当前对象实例（对于实例方法）或者类的Class对象（对于静态方法）。这意味着同一时间只有一个线程能够执行该对象的所有同步实例方法，或者该类的所有同步静态方法。
以下是`synchronized`方法如何保证线程安全的几个要点：

1. **互斥访问**：当一个线程正在执行一个对象的同步方法时，其他线程不能同时执行该对象的其他同步方法。这确保了共享数据的互斥访问。
2. **原子性**：同步方法中的操作是原子的，即在同一时刻，只有一个线程可以执行该方法内部的代码。
3. **内存可见性**：当一个线程进入同步方法时，它会清空工作内存中的共享变量值，从主内存中重新读取。当线程退出同步方法时，它会将工作内存中的共享变量的最新值刷新回主内存，从而保证了变量的可见性。
以下是一个使用`synchronized`方法的示例：
```java
public class Counter {
    private int count = 0;
    // 同步实例方法
    public synchronized void increment() {
        count++; // 这是一个复合操作，不是原子的
    }
    // 同步实例方法
    public synchronized int getCount() {
        return count;
    }
}
```
在这个例子中，`increment`和`getCount`方法都是同步的。这意味着当一个线程正在执行`increment`方法时，其他线程不能执行同一个`Counter`实例的`increment`或`getCount`方法，直到第一个线程完成`increment`方法的执行。
需要注意的是，虽然`synchronized`方法可以保证线程安全，但是过度使用同步可能会导致性能问题，因为它限制了并发性。因此，在设计并发程序时，应该尽量减少同步的范围和时间，只在必要时使用同步。

### transient

在Java中，`transient`关键字用于变量声明，表示该变量的值不应该被序列化。序列化是将对象状态转换为可存储或可传输的形式的过程，而反序列化则是将这种形式恢复为Java对象的过程。当你不希望某个字段被序列化时，你可以将其声明为`transient`。
以下是`transient`修饰符的一些用途：
1. **敏感数据保护**：如果你有一个包含敏感信息的字段，比如密码或密钥，你可能不希望这些信息在序列化对象时被保存到文件或通过网络发送。
2. **节省空间**：如果一个字段包含了大量数据，或者数据可以很容易地重新计算或检索，你可能不希望将其序列化，以减少序列化数据的大小。
3. **避免冗余**：如果一个字段是派生自其他字段或可以通过其他字段计算得出，那么序列化这个字段可能是多余的。
4. **自定义序列化**：有时，你可能需要自定义序列化过程，以便以特定的方式处理某些字段。在这种情况下，你可以将字段声明为`transient`，然后在类中实现`writeObject`和`readObject`方法来控制序列化和反序列化过程。
以下是一个使用`transient`关键字声明的字段示例：
```java
import java.io.Serializable;
public class User implements Serializable {
    private String name;
    private transient String password; // 不希望密码被序列化
    // 构造器、getter和setter省略
}
```
在这个例子中，`User`类的`password`字段被声明为`transient`，因此在序列化`User`对象时，`password`字段不会被包含在内。
需要注意的是，如果一个对象被声明为`transient`，那么在反序列化时，这个字段的值将被设置为该类型的默认值（例如，对于基本类型是0，对于对象类型是null）。因此，在设计类时，需要考虑到这一点，并确保在反序列化后正确地初始化`transient`字段。

### volatile

volatile 修饰的成员变量在每次被线程访问时，都强制从共享内存中重新读取该成员变量的值。而且，当成员变量发生变化时，会强制线程将变化值回写到共享内存。这样在任何时刻，两个不同的线程总是看到某个成员变量的同一个值。

一个 volatile 对象引用可能是 null。

```java
public class MyRunnable implements Runnable
{
    private volatile boolean active;
    public void run()
    {
        active = true;
        while (active) // 第一行
        {
            // 代码
        }
    }
    public void stop()
    {
        active = false; // 第二行
    }
}
```

通常情况下，在一个线程调用 run() 方法（在 Runnable 开启的线程），在另一个线程调用 stop() 方法。 如果 **第一行** 中缓冲区的 active 值被使用，那么在 **第二行** 的 active 值为 false 时循环不会停止。

但是以上代码中我们使用了 volatile 修饰 active，所以该循环会停止。

