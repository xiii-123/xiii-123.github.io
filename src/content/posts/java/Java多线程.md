---
title: Java多线程
published: 2025-03-15
description: 'Java中多线程竟然有这么多实现方法'
image: ''
tags: [java, 多线程]
category: 'java'
draft: false 
---
# 多线程实现方法
在Java中，实现多线程主要有以下三种方式：
1. **通过继承Thread类**
2. **通过实现Runnable接口**
3. **通过实现Callable接口和Future接口**
下面是每种方法的示例代码和注释：
## 1. 通过继承Thread类
```java
// 继承Thread类
class MyThread extends Thread {
    // 重写run方法
    @Override
    public void run() {
        System.out.println("线程运行中: " + Thread.currentThread().getName());
    }
}
public class ThreadExample {
    public static void main(String[] args) {
        // 创建线程对象
        MyThread myThread = new MyThread();
        // 启动线程
        myThread.start();
    }
}
```
## 2. 通过实现Runnable接口
```java
// 实现Runnable接口
class MyRunnable implements Runnable {
    // 实现run方法
    @Override
    public void run() {
        System.out.println("线程运行中: " + Thread.currentThread().getName());
    }
}
public class RunnableExample {
    public static void main(String[] args) {
        // 创建Runnable实例
        MyRunnable myRunnable = new MyRunnable();
        // 创建Thread对象，并将Runnable实例传递给Thread
        Thread thread = new Thread(myRunnable);
        // 启动线程
        thread.start();
    }
}
```
## 3. 通过实现Callable接口和Future接口
该方法是对上述两种方法的补充，可以获取多线程运行的结果。
- 实现`Callable`接口的`call`方法，即可带返回值
- 使用`FutureTask`对象即可获取多线程运行的返回值
```java
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;
// 实现Callable接口
class MyCallable implements Callable<String> {
    // 实现call方法
    @Override
    public String call() throws Exception {
        System.out.println("线程运行中: " + Thread.currentThread().getName());
        // 模拟耗时操作
        Thread.sleep(1000);
        return "Callable结果";
    }
}
public class CallableExample {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        // 创建Callable实例
        MyCallable myCallable = new MyCallable();
        // 使用FutureTask来包装Callable对象
        FutureTask<String> futureTask = new FutureTask<>(myCallable);
        // 创建Thread对象，并将FutureTask对象传递给Thread
        Thread thread = new Thread(futureTask);
        // 启动线程
        thread.start();
        
        // 获取线程执行的结果
        String result = futureTask.get();
        System.out.println("Callable线程返回的结果为: " + result);
    }
}
```
在上述示例中：
- 第一个示例通过继承`Thread`类并重写`run`方法来创建一个线程。
- 第二个示例通过实现`Runnable`接口并实现`run`方法来创建一个线程。
- 第三个示例通过实现`Callable`接口并实现`call`方法来创建一个线程，`Callable`接口可以返回结果，并且可以抛出异常。`FutureTask`是一个包装器，它实现了`RunnableFuture`接口，可以用来包装`Callable`或`Runnable`对象，并支持取消和获取结果的操作。

每种方法都有其用途和优势，通常推荐使用实现`Runnable`接口或`Callable`接口的方式，因为这样可以避免单继承的局限性，并且更加灵活。

# 线程优先级

在Java中，每个线程都有一个优先级，它可以帮助操作系统确定线程的调度顺序。线程优先级是一个介于`Thread.MIN_PRIORITY`和`Thread.MAX_PRIORITY`之间的整数，其中`Thread.MIN_PRIORITY`的值为1，而`Thread.MAX_PRIORITY`的值为10。默认情况下，每个线程都会继承其父线程的优先级，通常情况下，这个默认优先级是`Thread.NORM_PRIORITY`，其值为5。
以下是关于Java线程优先级的一些详细信息：
## 线程优先级的作用
线程优先级提供了一种建议，告诉调度器哪个线程相对更重要或更紧急。然而，实际的线程调度还取决于操作系统的策略和实现。不同的Java虚拟机实现和不同的操作系统可能会以不同的方式处理线程优先级。
## 设置和获取线程优先级
- `setPriority(int newPriority)`: 此方法用于设置线程的优先级。
- `getPriority()`: 此方法用于获取线程的优先级。
## 线程优先级的规则
- 优先级较高的线程在竞争资源时可能会比优先级较低的线程更有可能获得CPU时间。
- 优先级较高的线程并不保证一定比优先级较低的线程先执行，它只是提高了被首先执行的概率。
- 线程优先级可能会被忽略，特别是在非抢占式调度策略的系统中。
## 示例代码
以下是如何设置和获取线程优先级的示例：
```java
class MyThread extends Thread {
    public MyThread(String name) {
        super(name);
    }
    @Override
    public void run() {
        System.out.println("Running " + this.getName() + " with priority " + this.getPriority());
        // 执行一些任务...
    }
}
public class ThreadPriorityExample {
    public static void main(String[] args) {
        // 创建线程
        MyThread t1 = new MyThread("Thread-1");
        MyThread t2 = new MyThread("Thread-2");
        // 设置线程优先级
        t1.setPriority(Thread.MIN_PRIORITY); // 设置最低优先级
        t2.setPriority(Thread.MAX_PRIORITY); // 设置最高优先级
        // 启动线程
        t1.start();
        t2.start();
        // 获取线程优先级
        System.out.println("Thread-1 priority: " + t1.getPriority());
        System.out.println("Thread-2 priority: " + t2.getPriority());
    }
}
```
# 守护线程
在Java中，守护线程（Daemon Thread）是一种特殊的线程，它通常用于为其他线程提供服务，而不被视为程序中不可或缺的部分（俗称备胎线程）。

当所有的非守护线程结束时，即使守护线程仍然在运行，Java虚拟机（JVM）也会退出，所以守护线程也会强制退出。守护线程通常用于执行后台任务，如垃圾回收、JVM内部任务等。
以下是关于Java守护线程的详细介绍：
### 守护线程的特点
- **服务性**：守护线程通常用于执行系统级的后台任务，例如垃圾回收、JIT编译、事件处理等。
- **生命周期**：守护线程的生命周期依赖于JVM的生命周期。当JVM中只剩下守护线程时，JVM会退出。
- **创建**：默认情况下，线程不是守护线程，必须显式地将线程设置为守护线程。
- **异常处理**：如果JVM在守护线程中遇到未捕获的异常，JVM可能会默默地退出，而不会执行任何堆栈跟踪或错误报告。
### 设置守护线程
要设置一个线程为守护线程，可以在启动线程之前调用`setDaemon(true)`方法。以下是如何创建并设置守护线程的示例：
```java
public class DaemonThreadExample {
    public static void main(String[] args) {
        Thread daemonThread = new Thread(new DaemonRunnable());
        // 设置为守护线程
        daemonThread.setDaemon(true);
        // 启动守护线程
        daemonThread.start();
        
        // 主线程
        for (int i = 0; i < 10; i++) {
            System.out.println("Main thread is running...");
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("Main thread finished.");
    }
}
class DaemonRunnable implements Runnable {
    @Override
    public void run() {
        while (true) {
            System.out.println("Daemon thread is running...");
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```
在上面的例子中，`daemonThread`被设置为守护线程，它在主线程结束后会自动终止，即使守护线程的`run`方法中有一个无限循环。
### 注意事项
- 一旦线程启动，就不能更改其守护状态。尝试这样做会抛出`IllegalThreadStateException`。
- 守护线程不应该访问或修改重要的系统资源，因为它们可能在任何时刻被终止。
- 守护线程在创建子线程时，子线程默认也是守护线程。

# Thread.Yield和Thread.join*
## Thread.yield()
`Thread.yield()` 是一个静态方法，当当前线程调用它时，它暗示线程调度器当前线程愿意放弃当前对处理器的占用。这会导致线程从运行状态回到可运行状态，然后线程调度器可能会选择其他线程来运行。然而，这只是一个暗示，并不能保证线程调度器会立即选择其他线程来运行。
以下是一些关于 `Thread.yield()` 的要点：
- 它是一个静态方法，可以直接通过 `Thread` 类调用，不需要线程的实例。
- 它用于当前运行的线程。
- 它可能会导致当前线程暂停执行，允许具有相同优先级或更高优先级的其他线程运行。
- 它不会使当前线程进入等待或阻塞状态，线程仍然处于可运行状态。
- 使用 `Thread.yield()` 并不总是有明显的效果，因为线程调度器的具体行为依赖于Java虚拟机的实现和操作系统的线程调度策略。
示例代码：
```java
public class YieldExample extends Thread {
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(Thread.currentThread().getName() + " is running");
            Thread.yield();
        }
    }
    public static void main(String[] args) {
        YieldExample t1 = new YieldExample();
        YieldExample t2 = new YieldExample();
        t1.start();
        t2.start();
    }
}
```
## Thread.join()
`Thread.join()` 是一个实例方法，当在一个线程实例上调用它时，它允许一个线程等待另一个线程终止。换句话说，如果一个线程A调用了另一个线程B的 `join()` 方法，那么线程A将会暂停执行，直到线程B完成其任务。
以下是一些关于 `Thread.join()` 的要点：
- 它是一个实例方法，需要在特定的线程实例上调用。
- 它可以用来实现线程间的同步。
- 它可以抛出 `InterruptedException`，表示当前线程在等待时被中断。
- `join()` 方法有几种变体，如 `join(long millis)` 和 `join(long millis, int nanos)`，允许指定等待的最大时间。
示例代码：
```java
public class JoinExample {
    public static void main(String[] args) {
        Thread t1 = new Thread(new Runnable() {
            public void run() {
                for (int i = 0; i < 5; i++) {
                    System.out.println("Thread 1 is running");
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        });
        Thread t2 = new Thread(new Runnable() {
            public void run() {
                try {
                    // 等待线程t1完成
                    t1.join();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                for (int i = 0; i < 5; i++) {
                    System.out.println("Thread 2 is running");
                }
            }
        });
        t1.start();
        t2.start();
    }
}
```
在这个例子中，线程t2将等待线程t1完成其任务后才开始执行。

# 线程同步
### synchronized代码块
synchronized代码块是一种同步机制，它允许你指定一个锁对象。当线程进入synchronized代码块时，它会自动尝试获取锁对象上的锁。如果锁已被其他线程持有，则当前线程将等待，直到锁被释放。
#### 常用的锁对象：
1. **当前对象（this）**：在实例方法中使用synchronized代码块时，通常使用当前对象作为锁。
   
   ```java
   public void synchronizedBlock() {
       synchronized(this) {
           // 访问共享资源
       }
   }
   ```
2. **类对象（类名.class）**：当需要同步静态方法或静态变量时，通常使用类对象作为锁。
   
   ```java
   public void synchronizedStaticBlock() {
       synchronized(MyClass.class) {
           // 访问静态共享资源
       }
   }
   ```
3. **专门的锁对象**：可以创建一个专门的锁对象来作为多个synchronized代码块的锁。
   
   ```java
   private final Object lock = new Object();
   
   public void synchronizedCustomBlock() {
       synchronized(lock) {
           // 访问共享资源
       }
   }
   ```
#### 实践中的注意事项：
- **锁的选择**：选择锁对象时，应确保它是所有需要同步的代码块和方法的共享对象。通常，使用当前对象（this）或类对象（类名.class）是较好的选择，因为它们是唯一的。
### synchronized方法
synchronized方法是一种同步机制，其中整个方法体都是同步的。对于实例方法，锁是当前对象实例；对于静态方法，锁是类的Class对象。
#### 实践中的用法：
- **实例方法同步**：
  
  ```java
  public synchronized void synchronizedMethod() {
      // 访问共享资源
  }
  ```
- **静态方法同步**：
  
  ```java
  public static synchronized void synchronizedStaticMethod() {
      // 访问静态共享资源
  }
  ```
### Lock
Lock接口及其实现类（如ReentrantLock）提供了比synchronized更灵活的线程同步机制。它们允许显式地获取和释放锁，并且提供了尝试非阻塞地获取锁、支持中断的锁获取等功能。
#### 实践中的用法：
```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
public class LockExample {
    private final Lock lock = new ReentrantLock();
    public void lockedMethod() {
        lock.lock(); // 获取锁
        try {
            // 安全地访问共享资源
        } finally {
            lock.unlock(); // 释放锁
        }
    }
}
```
#### 实践中的注意事项：
- **锁的获取和释放**：始终在try-finally块中获取和释放锁，以确保即使发生异常也能释放锁。
- **条件变量**：Lock接口还支持条件变量（Condition），允许线程在某个条件下等待或被唤醒。

# 生产者消费者

让我们看一个经典的生产者消费者模式如何使用Java实现。

## 使用synchronized代码块
在这个例子中，我们使用一个共享的`LinkedList`作为缓冲区，生产者向其中添加元素，消费者从中移除元素。我们将使用`wait()`和`notifyAll()`方法来处理线程间的同步。
```java
import java.util.LinkedList;
public class ProducerConsumerExample {
    public static void main(String[] args) {
        LinkedList<Integer> buffer = new LinkedList<>();
        int capacity = 10; // 缓冲区容量
        Thread producerThread = new Thread(new Producer(buffer, capacity), "Producer");
        Thread consumerThread = new Thread(new Consumer(buffer), "Consumer");
        producerThread.start();
        consumerThread.start();
    }
}
class Producer implements Runnable {
    private LinkedList<Integer> buffer;
    private int capacity;
    public Producer(LinkedList<Integer> buffer, int capacity) {
        this.buffer = buffer;
        this.capacity = capacity;
    }
    @Override
    public void run() {
        int value = 0;
        while (true) {
            synchronized (buffer) {
                while (buffer.size() == capacity) {
                    try {
                        buffer.wait(); // 等待缓冲区有空间
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println(Thread.currentThread().getName() + " produced: " + value);
                buffer.add(value++);
                buffer.notifyAll(); // 通知消费者可以消费了
            }
        }
    }
}
class Consumer implements Runnable {
    private LinkedList<Integer> buffer;
    public Consumer(LinkedList<Integer> buffer) {
        this.buffer = buffer;
    }
    @Override
    public void run() {
        while (true) {
            synchronized (buffer) {
                while (buffer.isEmpty()) {
                    try {
                        buffer.wait(); // 等待缓冲区有数据
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                int value = buffer.removeFirst();
                System.out.println(Thread.currentThread().getName() + " consumed: " + value);
                buffer.notifyAll(); // 通知生产者可以生产了
            }
        }
    }
}
```

## 使用阻塞队列
在Java中，可以使用`java.util.concurrent.BlockingQueue`接口来实现生产者消费者模型。阻塞队列提供了线程安全的队列操作，并且当队列满时，插入操作会阻塞；当队列空时，获取操作会阻塞。以下是一个使用`ArrayBlockingQueue`实现的生产者消费者模型的示例代码：
```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;
public class ProducerConsumerWithBlockingQueue {
    public static void main(String[] args) {
        BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(10); // 创建一个容量为10的阻塞队列
        Thread producerThread = new Thread(new Producer(queue), "Producer");
        Thread consumerThread = new Thread(new Consumer(queue), "Consumer");
        producerThread.start();
        consumerThread.start();
    }
    static class Producer implements Runnable {
        private final BlockingQueue<Integer> queue;
        public Producer(BlockingQueue<Integer> queue) {
            this.queue = queue;
        }
        @Override
        public void run() {
            try {
                for (int i = 0; i < 20; i++) {
                    System.out.println(Thread.currentThread().getName() + " produced: " + i);
                    queue.put(i); // 将元素插入队列，如果队列满，则阻塞
                    Thread.sleep(100); // 模拟生产时间
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
    static class Consumer implements Runnable {
        private final BlockingQueue<Integer> queue;
        public Consumer(BlockingQueue<Integer> queue) {
            this.queue = queue;
        }
        @Override
        public void run() {
            try {
                while (true) {
                    Integer item = queue.take(); // 从队列中获取元素，如果队列为空，则阻塞
                    System.out.println(Thread.currentThread().getName() + " consumed: " + item);
                    Thread.sleep(100); // 模拟消费时间
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
}
```
在这个示例中：
- `Producer`类负责生产数据，并将数据放入阻塞队列中。如果队列已满，`put()`方法会阻塞，直到队列中有空间。
- `Consumer`类负责从阻塞队列中取出数据。如果队列为空，`take()`方法会阻塞，直到队列中有数据可以取出。
- `ArrayBlockingQueue`是一个有界阻塞队列，其容量在创建时指定。

使用阻塞队列可以简化生产者消费者模型的实现，因为它自动处理了线程同步和等待通知机制，从而避免了手动使用`wait()`和`notifyAll()`的需要。

# 线程池
Java中的线程池是一种用于管理和重用线程的机制，它可以提高性能并减少创建线程和销毁线程的开销。以下是对Java线程池的介绍：
## 1. Executor 提供的默认创建线程池
Java中的`java.util.concurrent`包提供了`Executors`类，它包含一些静态工厂方法来创建线程池。以下是几个常用的方法：
- `Executors.newCachedThreadPool()`：创建一个可根据需要创建新线程的线程池，但会在先前构建的线程可用时重用它们。这个线程池在执行很多短期异步任务时很有用。
- `Executors.newFixedThreadPool(int nThreads)`：创建一个可重用固定数量线程的线程池。如果所有线程都处于活动状态，则提交的新任务将在队列中等待，直到有线程可用。
- `Executors.newSingleThreadExecutor()`：创建一个单线程的Executor，确保所有任务都在同一线程中按顺序执行。
- `Executors.newScheduledThreadPool(int corePoolSize)`：创建一个可定时执行或周期性执行任务的线程池。
## 2. 自定义线程池
`ThreadPoolExecutor` 构造函数有七个参数。以下是完整的参数列表及其说明：
1. `int corePoolSize`：线程池中的核心线程数，即使它们是空闲的，这些线程也会保持活跃状态。
2. `int maximumPoolSize`：线程池中允许的最大线程数。
3. `long keepAliveTime`：当线程数大于核心线程数时，这是多余空闲线程在终止前等待新任务的最长时间。
4. `TimeUnit unit`：`keepAliveTime` 参数的时间单位。
5. `BlockingQueue<Runnable> workQueue`：用于在执行任务之前保存任务的队列。这个队列仅保存由 `execute` 方法提交的 `Runnable` 任务。
6. `ThreadFactory threadFactory`：执行程序创建新线程时使用的工厂。
7. `RejectedExecutionHandler handler`：当队列满并且线程数达到最大线程数时，用于处理无法执行的任务的处理程序。
以下是一个包含所有参数的 `ThreadPoolExecutor` 示例：
```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.RejectedExecutionHandler;
import java.util.concurrent.Executors;
public class CustomThreadPoolExample {
    public static void main(String[] args) {
        int corePoolSize = 5;
        int maximumPoolSize = 10;
        long keepAliveTime = 5000;
        TimeUnit unit = TimeUnit.MILLISECONDS;
        BlockingQueue<Runnable> workQueue = new ArrayBlockingQueue<>(100);
        ThreadFactory threadFactory = Executors.defaultThreadFactory();
        RejectedExecutionHandler handler = new ThreadPoolExecutor.CallerRunsPolicy(); // 一种拒绝策略
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
                corePoolSize,
                maximumPoolSize,
                keepAliveTime,
                unit,
                workQueue,
                threadFactory,
                handler
        );
        // 提交任务到线程池
        for (int i = 0; i < 20; i++) {
            executor.execute(new Task());
        }
        // 关闭线程池
        executor.shutdown();
    }
    static class Task implements Runnable {
        @Override
        public void run() {
            System.out.println("Executing task: " + Thread.currentThread().getName());
            // 执行具体任务
        }
    }
}
```
在这个例子中，我们使用了 `ThreadPoolExecutor.CallerRunsPolicy` 作为拒绝策略，这意味着当任务无法被线程池接受时，提交任务的线程将尝试自己执行该任务。还有其他几种拒绝策略，如 `AbortPolicy`（抛出 `RejectedExecutionException`）、`DiscardPolicy`（默默丢弃无法处理的任务）和 `DiscardOldestPolicy`（丢弃队列中最旧的任务，然后重试执行）。

## `submit`和`execute`
在Java的`ExecutorService`接口中，`submit`和`execute`是两个用于提交任务执行的方法，它们之间有一些关键的区别：
1. **返回值**:
   - `submit`方法返回一个`Future`对象，该对象可以用来检查任务是否已完成，等待任务完成，并获取任务的结果（如果任务有返回结果的话）。这对于需要异步处理并获取处理结果的情况非常有用。
   - `execute`方法没有返回值。它只是简单地执行任务，不提供任何关于任务状态或结果的信息。
2. **异常处理**:
   - 当使用`submit`方法提交任务时，如果任务在执行过程中抛出异常，那么这个异常可以被`Future.get()`方法捕获，或者在任务完成后通过`Future.isDone()`和`Future.cancel()`来处理。
   - 使用`execute`方法提交的任务，如果抛出未捕获的异常，则默认情况下会由线程池中的线程处理，即异常会被打印到控制台并终止该线程。可以通过为线程设置一个未捕获异常处理器（`UncaughtExceptionHandler`）来捕获这些异常。
3. **任务类型**:
   - `submit`方法可以接受`Callable`和`Runnable`任务。`Callable`任务可以返回结果，而`Runnable`任务不能。
   - `execute`方法只能接受`Runnable`任务。
以下是一个简单的示例，展示了如何使用`submit`和`execute`：
```java
import java.util.concurrent.*;
public class SubmitExecuteExample {
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        ExecutorService executorService = Executors.newFixedThreadPool(2);
        // 使用execute提交Runnable任务
        executorService.execute(() -> System.out.println("Runnable task executed by execute."));
        // 使用submit提交Callable任务
        Future<String> future = executorService.submit(() -> {
            System.out.println("Callable task executed by submit.");
            return "Callable result";
        });
        // 获取Callable任务的结果
        String result = future.get();
        System.out.println("Result of callable task: " + result);
        // 关闭ExecutorService
        executorService.shutdown();
    }
}
```
在这个例子中，我们首先使用`execute`提交了一个`Runnable`任务，然后使用`submit`提交了一个`Callable`任务，并通过返回的`Future`对象获取了任务的结果。
