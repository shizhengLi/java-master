# 并发编程的艺术：从synchronized到Disruptor

## 引言

并发编程是Java技术体系中最具挑战性的领域之一。从基础的synchronized关键字到高性能的Disruptor框架，Java并发编程技术经历了漫长的发展历程。在这篇文章中，我将深入探讨Java并发编程的核心概念、实现原理、性能优化以及最佳实践。

## 1. 并发编程基础概念

### 1.1 并发与并行的区别

```java
public class ConcurrencyBasics {
    // 并发（Concurrency）和并行（Parallelism）的区别

    public void explainConcurrency() {
        // 并发：多个任务在同一时间段内交替执行
        // 通过时间片轮转实现，给用户一种同时执行的假象

        // 并行：多个任务在同一时刻真正同时执行
        // 需要多核CPU支持

        // 举例说明：
        // 单核CPU运行多个程序：并发
        // 多核CPU运行多个程序：可以并行

        // Java中的并发既包含时间片轮转的并发
        // 也包含多核CPU下的并行执行
    }

    public void javaConcurrencyModel() {
        // Java的并发模型基于：
        // 1. 线程（Thread）
        // 2. 共享内存
        // 3. 同步机制

        // 线程是Java并发编程的基本单位
        // 所有线程共享JVM的堆内存
        // 通过同步机制保证线程安全
    }
}
```

### 1.2 线程的生命周期

```java
public class ThreadLifecycle {
    public static void main(String[] args) {
        // 线程的六种状态：
        // 1. NEW: 新创建的线程
        // 2. RUNNABLE: 可运行状态（包含运行中和就绪）
        // 3. BLOCKED: 阻塞状态（等待锁）
        // 4. WAITING: 等待状态（无限期等待）
        // 5. TIMED_WAITING: 限时等待状态
        // 6. TERMINATED: 终止状态

        Thread thread = new Thread(() -> {
            // 线程体
            System.out.println("Thread is running");
        });

        // NEW状态
        System.out.println("Thread state: " + thread.getState());

        thread.start(); // 进入RUNNABLE状态
        System.out.println("Thread state after start: " + thread.getState());

        try {
            thread.join(); // 等待线程结束
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("Thread state after completion: " + thread.getState());
    }
}
```

## 2. synchronized关键字深度解析

### 2.1 synchronized的基本用法

```java
public class SynchronizedBasics {
    private int count = 0;
    private final Object lock = new Object();

    // 1. 修饰实例方法
    public synchronized void increment() {
        count++;
    }

    // 2. 修饰静态方法
    public static synchronized void staticMethod() {
        // 同步的是类对象
    }

    // 3. 修饰代码块
    public void blockSynchronization() {
        synchronized (this) {
            // 同步当前实例
            count++;
        }

        synchronized (SynchronizedBasics.class) {
            // 同步类对象
        }

        synchronized (lock) {
            // 同步指定对象
            count++;
        }
    }

    // synchronized的底层原理：
    // 1. 基于Monitor对象实现
    // 2. 通过monitorenter和monitorexit指令控制
    // 3. 包含偏向锁、轻量级锁、重量级锁的升级过程
}
```

### 2.2 synchronized的锁升级过程

```java
public class LockUpgrade {
    private final Object lock = new Object();

    public void demonstrateLockUpgrade() {
        synchronized (lock) {
            // 锁升级过程：
            // 1. 偏向锁：当只有一个线程访问时
            // 2. 轻量级锁：当有竞争时，尝试自旋
            // 3. 重量级锁：自旋失败后，升级为重量级锁

            // 偏向锁（Biased Locking）：
            // - 假设锁总是由同一个线程获取
            // - 避免CAS操作，提高性能
            // - 通过mark word中的偏向线程ID实现

            // 轻量级锁（Lightweight Locking）：
            // - 线程尝试通过CAS获取锁
            // - 失败时会自旋一定次数
            // - 适用于短时间持有的锁

            // 重量级锁（Heavyweight Locking）：
            // - 涉及操作系统层面的互斥量
            // - 线程阻塞，不消耗CPU
            // - 适用于长时间持有的锁
        }
    }

    // 相关JVM参数：
    // -XX:+UseBiasedLocking 启用偏向锁（默认启用）
    // -XX:BiasedLockingStartupDelay 偏向锁启动延迟
    // -XX:PreBlockSpin 自旋次数
}
```

### 2.3 synchronized的优化

```java
public class SynchronizedOptimization {
    // 1. 锁消除（Lock Elimination）
    public void lockElimination() {
        // JIT编译器可以消除不必要的同步
        Object obj = new Object();
        synchronized (obj) {
            // obj对象不逸出，同步可以消除
            System.out.println("Lock elimination example");
        }
    }

    // 2. 锁粗化（Lock Coarsening）
    public void lockCoarsening() {
        // JIT编译器会将邻近的同步块合并
        for (int i = 0; i < 100; i++) {
            synchronized (this) {
                // 多次同步会被粗化为一个
                count++;
            }
        }
        // 优化为：
        synchronized (this) {
            for (int i = 0; i < 100; i++) {
                count++;
            }
        }
    }

    // 3. 自适应自旋（Adaptive Spinning）
    public void adaptiveSpinning() {
        // JVM会根据历史情况调整自旋次数
        // 如果之前自旋成功，增加自旋次数
        // 如果之前自旋失败，减少自旋次数
    }

    private int count = 0;
}
```

## 3. volatile关键字深入理解

### 3.1 volatile的特性

```java
public class VolatileFeatures {
    // volatile的两个主要特性：
    // 1. 可见性（Visibility）
    // 2. 禁止指令重排序（Ordering）

    private volatile boolean flag = false;
    private int value = 0;

    public void writer() {
        value = 42;        // 1
        flag = true;       // 2
    }

    public void reader() {
        if (flag) {        // 3
            int result = value; // 4
            System.out.println("Result: " + result);
        }
    }

    // volatile的内存语义：
    // - 写操作：插入StoreStore和StoreLoad屏障
    // - 读操作：插入LoadLoad和LoadStore屏障
    // - 确保变量的可见性和有序性
}
```

### 3.2 volatile的实现原理

```java
public class VolatileImplementation {
    private volatile int sharedValue = 0;

    public void demonstrateVolatile() {
        // volatile的实现基于：
        // 1. 内存屏障（Memory Barrier）
        // 2. happens-before原则
        // 3. MESI缓存一致性协议

        // 内存屏障类型：
        // - LoadLoad: Load1; LoadLoad; Load2
        // - StoreStore: Store1; StoreStore; Store2
        // - LoadStore: Load1; LoadStore; Store2
        // - StoreLoad: Store1; StoreLoad; Load2

        // volatile的适用场景：
        // 1. 状态标志位
        // 2. 一次性安全发布
        // 3. 独立观察
        // 4. "volatile bean"模式
    }
}
```

## 4. java.util.concurrent包详解

### 4.1 原子类（Atomic Classes）

```java
import java.util.concurrent.atomic.*;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class AtomicClasses {
    // 1. 基本类型原子类
    private AtomicInteger atomicInt = new AtomicInteger(0);
    private AtomicLong atomicLong = new AtomicLong(0);
    private AtomicBoolean atomicBoolean = new AtomicBoolean(false);

    // 2. 引用类型原子类
    private AtomicReference<String> atomicRef = new AtomicReference<>("initial");

    // 3. 数组类型原子类
    private AtomicIntegerArray atomicIntArray = new AtomicIntegerArray(10);

    // 4. 字段更新器
    private static class User {
        volatile int age;
        volatile String name;
    }

    private AtomicIntegerFieldUpdater<User> ageUpdater =
        AtomicIntegerFieldUpdater.newUpdater(User.class, "age");

    public void demonstrateAtomic() {
        // 基本操作
        atomicInt.incrementAndGet();  // 原子自增
        atomicInt.compareAndSet(1, 2); // CAS操作
        atomicInt.getAndAdd(5);        // 获取并增加

        // 引用类型操作
        atomicRef.compareAndSet("old", "new");

        // 字段更新
        User user = new User();
        ageUpdater.incrementAndGet(user);
    }

    // 原子类的实现原理：
    // 1. 基于Unsafe类的CAS操作
    // 2. 底层使用CPU的CAS指令
    // 3. 避免了锁的开销

    // 原子类的性能特点：
    // - 在低竞争情况下性能很好
    // - 在高竞争情况下可能产生大量CAS失败
    // - 适合读多写少的场景
}
```

### 4.2 锁（Locks）

```java
import java.util.concurrent.locks.*;
import java.util.concurrent.TimeUnit;

public class LockMechanisms {
    // 1. ReentrantLock
    private final ReentrantLock reentrantLock = new ReentrantLock();
    private final ReentrantLock fairLock = new ReentrantLock(true);

    public void reentrantLockExample() {
        reentrantLock.lock();
        try {
            // 临界区代码
            System.out.println("ReentrantLock example");
        } finally {
            reentrantLock.unlock();
        }

        // 公平锁与非公平锁
        // - 公平锁：按请求顺序获取锁
        // - 非公平锁：可以插队，吞吐量更高

        // ReentrantLock的特性：
        // - 可重入
        // - 可中断
        // - 可超时
        // - 公平性选择
    }

    // 2. ReadWriteLock
    private final ReadWriteLock readWriteLock = new ReentrantReadWriteLock();
    private final Lock readLock = readWriteLock.readLock();
    private final Lock writeLock = readWriteLock.writeLock();

    public void readWriteLockExample() {
        // 读操作
        readLock.lock();
        try {
            // 并发读取
            System.out.println("Reading data");
        } finally {
            readLock.unlock();
        }

        // 写操作
        writeLock.lock();
        try {
            // 独占写入
            System.out.println("Writing data");
        } finally {
            writeLock.unlock();
        }

        // ReadWriteLock的特性：
        // - 读锁共享，写锁独占
        // - 读锁可以同时被多个线程持有
        // - 写锁只能被一个线程持有
        // - 适合读多写少的场景
    }

    // 3. StampedLock
    private final StampedLock stampedLock = new StampedLock();

    public void stampedLockExample() {
        // 乐观读
        long stamp = stampedLock.tryOptimisticRead();
        try {
            // 读取数据
            // 如果数据被修改，转换为悲观读
            if (!stampedLock.validate(stamp)) {
                stamp = stampedLock.readLock();
                try {
                    // 重新读取
                } finally {
                    stampedLock.unlockRead(stamp);
                }
            }
        } finally {
            if (stampedLock.validate(stamp)) {
                stampedLock.unlock(stamp);
            }
        }

        // StampedLock的特性：
        // - 三种模式：写、读、乐观读
        // - 乐观读不阻塞写操作
        // - 性能比ReadWriteLock更好
        // - 但实现更复杂
    }
}
```

### 4.3 并发集合（Concurrent Collections）

```java
import java.util.concurrent.*;

public class ConcurrentCollections {
    // 1. ConcurrentHashMap
    private final ConcurrentHashMap<String, Integer> concurrentMap =
        new ConcurrentHashMap<>();

    public void concurrentHashMapExample() {
        // 线程安全的操作
        concurrentMap.put("key1", 1);
        concurrentMap.putIfAbsent("key2", 2);
        concurrentMap.compute("key3", (k, v) -> v == null ? 3 : v + 1);

        // 原子操作
        concurrentMap.merge("key4", 1, Integer::sum);

        // ConcurrentHashMap的实现原理：
        // - 分段锁（Java 7）或CAS + synchronized（Java 8+）
        // - 并发度可以通过构造函数设置
        // - 适合高并发场景
    }

    // 2. CopyOnWriteArrayList
    private final CopyOnWriteArrayList<String> copyOnWriteList =
        new CopyOnWriteArrayList<>();

    public void copyOnWriteExample() {
        // 写操作会复制底层数组
        copyOnWriteList.add("element1");
        copyOnWriteList.add("element2");

        // 读操作不需要同步
        for (String element : copyOnWriteList) {
            System.out.println(element);
        }

        // CopyOnWrite的特性：
        // - 写时复制
        // - 读操作性能好
        - 写操作成本高
        - 适合读多写少的场景
    }

    // 3. ConcurrentLinkedQueue
    private final ConcurrentLinkedQueue<String> concurrentQueue =
        new ConcurrentLinkedQueue<>();

    public void concurrentQueueExample() {
        // 无锁并发队列
        concurrentQueue.offer("item1");
        concurrentQueue.offer("item2");

        String item = concurrentQueue.poll();

        // ConcurrentLinkedQueue的特性：
        // - 基于CAS实现
        - 无阻塞算法
        - 高并发性能好
    }

    // 4. BlockingQueue
    private final BlockingQueue<String> blockingQueue =
        new LinkedBlockingQueue<>(10);

    public void blockingQueueExample() {
        try {
            // 阻塞操作
            blockingQueue.put("item1");  // 如果队列满，阻塞
            String item = blockingQueue.take(); // 如果队列空，阻塞

            // 非阻塞操作
            boolean success = blockingQueue.offer("item2");
            item = blockingQueue.poll();

            // 限时操作
            success = blockingQueue.offer("item3", 1, TimeUnit.SECONDS);
            item = blockingQueue.poll(1, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```

### 4.4 线程池（Thread Pools）

```java
import java.util.concurrent.*;

public class ThreadPoolManagement {
    // 1. 基础线程池
    private final ExecutorService fixedThreadPool =
        Executors.newFixedThreadPool(10);

    private final ExecutorService cachedThreadPool =
        Executors.newCachedThreadPool();

    private final ExecutorService singleThreadExecutor =
        Executors.newSingleThreadExecutor();

    // 2. 推荐使用ThreadPoolExecutor
    private final ThreadPoolExecutor threadPoolExecutor =
        new ThreadPoolExecutor(
            5,                      // 核心线程数
            10,                     // 最大线程数
            60L,                    // 空闲线程存活时间
            TimeUnit.SECONDS,       // 时间单位
            new LinkedBlockingQueue<>(100), // 工作队列
            Executors.defaultThreadFactory(), // 线程工厂
            new ThreadPoolExecutor.AbortPolicy() // 拒绝策略
        );

    public void threadPoolExample() {
        // 提交任务
        threadPoolExecutor.execute(() -> {
            System.out.println("Task executed");
        });

        // 提交有返回值的任务
        Future<String> future = threadPoolExecutor.submit(() -> {
            return "Task result";
        });

        try {
            String result = future.get();
            System.out.println("Result: " + result);
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }

        // 线程池的核心参数：
        // - 核心线程数：长期存活的线程数量
        // - 最大线程数：线程池能创建的最大线程数
        // - 工作队列：存放待执行任务的队列
        // - 拒绝策略：队列满时的处理策略

        // 线程池的工作原理：
        // 1. 如果核心线程数未满，创建新线程
        // 2. 如果核心线程数满，任务加入队列
        // 3. 如果队列满且未达到最大线程数，创建新线程
        // 4. 如果达到最大线程数，执行拒绝策略
    }

    // 3. 定时任务线程池
    private final ScheduledExecutorService scheduledExecutor =
        Executors.newScheduledThreadPool(5);

    public void scheduledExecutorExample() {
        // 延迟执行
        scheduledExecutor.schedule(() -> {
            System.out.println("Delayed task");
        }, 1, TimeUnit.SECONDS);

        // 固定频率执行
        scheduledExecutor.scheduleAtFixedRate(() -> {
            System.out.println("Fixed rate task");
        }, 0, 1, TimeUnit.SECONDS);

        // 固定延迟执行
        scheduledExecutor.scheduleWithFixedDelay(() -> {
            System.out.println("Fixed delay task");
        }, 0, 1, TimeUnit.SECONDS);
    }

    // 4. ForkJoinPool
    private final ForkJoinPool forkJoinPool = new ForkJoinPool();

    public void forkJoinExample() {
        // ForkJoinTask
        RecursiveTask<Integer> task = new RecursiveTask<Integer>() {
            @Override
            protected Integer compute() {
                // 任务分解和结果合并
                if (taskSize < THRESHOLD) {
                    return computeDirectly();
                }

                RecursiveTask<Integer> leftTask = new MyTask(leftHalf);
                RecursiveTask<Integer> rightTask = new MyTask(rightHalf);

                leftTask.fork();
                rightTask.fork();

                return leftTask.join() + rightTask.join();
            }
        };

        int result = forkJoinPool.invoke(task);
        System.out.println("ForkJoin result: " + result);
    }

    private static final int THRESHOLD = 1000;
    private int taskSize = 10000;

    private int computeDirectly() {
        // 直接计算
        return 0;
    }
}
```

## 5. 并发编程高级主题

### 5.1 ThreadLocal深入解析

```java
public class ThreadLocalDeepDive {
    // ThreadLocal的内部结构
    private static final ThreadLocal<String> threadLocal = new ThreadLocal<>();

    public void demonstrateThreadLocal() {
        // 设置值
        threadLocal.set("value1");

        // 获取值
        String value = threadLocal.get();

        // 删除值
        threadLocal.remove();

        // ThreadLocal的实现原理：
        // 1. 每个Thread对象中有一个ThreadLocalMap
        // 2. ThreadLocalMap使用ThreadLocal作为key
        // 3. 值存储在当前线程的ThreadLocalMap中
        // 4. 实现了线程间的数据隔离
    }

    // ThreadLocal的内存泄漏问题
    public void memoryLeakExample() {
        // ThreadLocal可能导致内存泄漏
        // 1. ThreadLocal的key是弱引用
        // 2. value是强引用
        // 3. 如果ThreadLocal被回收，key变为null
        // 4. 但value仍然存在，无法被回收

        // 解决方案：
        // 1. 使用后调用remove()方法
        // 2. 使用try-finally确保清理
        try {
            threadLocal.set("temp value");
            // 使用ThreadLocal
        } finally {
            threadLocal.remove(); // 确保清理
        }
    }

    // InheritableThreadLocal
    private static final InheritableThreadLocal<String> inheritableThreadLocal =
        new InheritableThreadLocal<>();

    public void inheritableThreadLocalExample() {
        // InheritableThreadLocal可以在子线程中继承父线程的值
        inheritableThreadLocal.set("parent value");

        new Thread(() -> {
            String childValue = inheritableThreadLocal.get();
            System.out.println("Child thread value: " + childValue);
        }).start();
    }
}
```

### 5.2 CompletableFuture异步编程

```java
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;

public class CompletableFutureProgramming {
    public void basicCompletableFuture() {
        // 1. 创建CompletableFuture
        CompletableFuture<String> future = new CompletableFuture<>();

        // 2. 异步完成
        new Thread(() -> {
            try {
                Thread.sleep(1000);
                future.complete("Result from async task");
            } catch (InterruptedException e) {
                future.completeExceptionally(e);
            }
        }).start();

        // 3. 处理结果
        future.thenAccept(result -> {
            System.out.println("Result: " + result);
        });

        future.exceptionally(ex -> {
            System.out.println("Exception: " + ex.getMessage());
            return null;
        });
    }

    public void supplyAsyncExample() {
        // 1. 异步执行任务
        CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
            // 模拟耗时操作
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            return "Async result";
        });

        // 2. 链式操作
        future.thenApply(result -> {
            return result.toUpperCase();
        }).thenAccept(upperResult -> {
            System.out.println("Upper result: " + upperResult);
        });

        // 3. 组合操作
        CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> "Hello");
        CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "World");

        future1.thenCombine(future2, (s1, s2) -> s1 + " " + s2)
               .thenAccept(combined -> System.out.println("Combined: " + combined));

        // 4. 异常处理
        CompletableFuture.supplyAsync(() -> {
            throw new RuntimeException("Something went wrong");
        }).exceptionally(ex -> {
            System.out.println("Handled exception: " + ex.getMessage());
            return "Fallback value";
        }).thenAccept(result -> System.out.println("Final result: " + result));
    }

    public void advancedCompletableFuture() {
        // 1. 多任务组合
        CompletableFuture<Void> allFutures = CompletableFuture.allOf(
            CompletableFuture.supplyAsync(() -> "Task 1"),
            CompletableFuture.supplyAsync(() -> "Task 2"),
            CompletableFuture.supplyAsync(() -> "Task 3")
        );

        allFutures.thenRun(() -> {
            System.out.println("All tasks completed");
        });

        // 2. 任意任务完成
        CompletableFuture<Object> anyFuture = CompletableFuture.anyOf(
            CompletableFuture.supplyAsync(() -> "Fast task"),
            CompletableFuture.supplyAsync(() -> {
                try {
                    Thread.sleep(2000);
                    return "Slow task";
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    return null;
                }
            })
        );

        anyFuture.thenAccept(result -> {
            System.out.println("First completed: " + result);
        });

        // 3. 超时处理
        CompletableFuture<String> timeoutFuture = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(2000);
                return "Delayed result";
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return null;
            }
        });

        timeoutFuture.orTimeout(1, TimeUnit.SECONDS)
                   .exceptionally(ex -> {
                       System.out.println("Timeout occurred");
                       return "Timeout fallback";
                   });

        // 4. 自定义线程池
        ExecutorService customExecutor = Executors.newFixedThreadPool(5);
        CompletableFuture.supplyAsync(() -> "Custom executor task", customExecutor)
                       .thenAccept(result -> System.out.println(result));
    }
}
```

### 5.3 并发编程模式

```java
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class ConcurrentPatterns {
    // 1. 生产者-消费者模式
    private final BlockingQueue<Integer> queue = new LinkedBlockingQueue<>(10);
    private final AtomicInteger counter = new AtomicInteger(0);

    public void producerConsumerPattern() {
        // 生产者
        Runnable producer = () -> {
            try {
                while (true) {
                    int value = counter.incrementAndGet();
                    queue.put(value);
                    System.out.println("Produced: " + value);
                    Thread.sleep(100);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        };

        // 消费者
        Runnable consumer = () -> {
            try {
                while (true) {
                    int value = queue.take();
                    System.out.println("Consumed: " + value);
                    Thread.sleep(200);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        };

        // 启动生产者和消费者
        new Thread(producer).start();
        new Thread(consumer).start();
    }

    // 2. 读写锁模式
    private final ReadWriteLock rwLock = new ReentrantReadWriteLock();
    private final Lock readLock = rwLock.readLock();
    private final Lock writeLock = rwLock.writeLock();
    private final Map<String, String> data = new ConcurrentHashMap<>();

    public void readWriteLockPattern() {
        // 读取操作
        Runnable reader = () -> {
            readLock.lock();
            try {
                System.out.println("Reading data: " + data.size());
                Thread.sleep(100);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                readLock.unlock();
            }
        };

        // 写入操作
        Runnable writer = () -> {
            writeLock.lock();
            try {
                String key = "key" + System.currentTimeMillis();
                data.put(key, "value");
                System.out.println("Written: " + key);
                Thread.sleep(50);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                writeLock.unlock();
            }
        };

        // 启动多个读写线程
        for (int i = 0; i < 5; i++) {
            new Thread(reader).start();
        }
        for (int i = 0; i < 2; i++) {
            new Thread(writer).start();
        }
    }

    // 3. 线程池隔离模式
    private final ExecutorService ioThreadPool =
        Executors.newFixedThreadPool(10);
    private final ExecutorService cpuThreadPool =
        Executors.newFixedThreadPool(Runtime.getRuntime().availableProcessors());

    public void threadPoolIsolationPattern() {
        // I/O密集型任务
        ioThreadPool.submit(() -> {
            // 处理I/O操作
            try {
                Thread.sleep(1000); // 模拟I/O等待
                System.out.println("I/O task completed");
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });

        // CPU密集型任务
        cpuThreadPool.submit(() -> {
            // 处理CPU密集型计算
            long sum = 0;
            for (int i = 0; i < 1000000; i++) {
                sum += i;
            }
            System.out.println("CPU task completed, sum: " + sum);
        });
    }

    // 4. Future模式
    public void futurePattern() {
        ExecutorService executor = Executors.newFixedThreadPool(5);

        // 提交多个异步任务
        Future<String> future1 = executor.submit(() -> {
            Thread.sleep(1000);
            return "Result 1";
        });

        Future<String> future2 = executor.submit(() -> {
            Thread.sleep(1500);
            return "Result 2";
        });

        Future<String> future3 = executor.submit(() -> {
            Thread.sleep(800);
            return "Result 3";
        });

        // 处理结果
        try {
            System.out.println("Future 1: " + future1.get());
            System.out.println("Future 2: " + future2.get());
            System.out.println("Future 3: " + future3.get());
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }

        executor.shutdown();
    }

    // 5. CountDownLatch模式
    public void countDownLatchPattern() {
        final int threadCount = 3;
        final CountDownLatch latch = new CountDownLatch(threadCount);

        for (int i = 0; i < threadCount; i++) {
            new Thread(() -> {
                try {
                    System.out.println("Thread started");
                    Thread.sleep(1000);
                    System.out.println("Thread completed");
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    latch.countDown();
                }
            }).start();
        }

        try {
            latch.await(); // 等待所有线程完成
            System.out.println("All threads completed");
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }

    // 6. CyclicBarrier模式
    public void cyclicBarrierPattern() {
        final int threadCount = 3;
        final CyclicBarrier barrier = new CyclicBarrier(threadCount, () -> {
            System.out.println("All threads reached barrier");
        });

        for (int i = 0; i < threadCount; i++) {
            new Thread(() -> {
                try {
                    System.out.println("Thread started");
                    Thread.sleep(new Random().nextInt(2000));
                    System.out.println("Thread waiting at barrier");
                    barrier.await();
                    System.out.println("Thread passed barrier");
                } catch (InterruptedException | BrokenBarrierException e) {
                    Thread.currentThread().interrupt();
                }
            }).start();
        }
    }
}
```

## 6. 高性能并发框架

### 6.1 Disruptor框架

```java
import com.lmax.disruptor.*;
import com.lmax.disruptor.dsl.Disruptor;
import com.lmax.disruptor.dsl.ProducerType;

public class DisruptorFramework {
    // 1. 事件类
    public static class ValueEvent {
        private long value;

        public long getValue() { return value; }
        public void setValue(long value) { this.value = value; }
    }

    // 2. 事件工厂
    public static class ValueEventFactory implements EventFactory<ValueEvent> {
        @Override
        public ValueEvent newInstance() {
            return new ValueEvent();
        }
    }

    // 3. 事件处理器
    public static class ValueEventHandler implements EventHandler<ValueEvent> {
        @Override
        public void onEvent(ValueEvent event, long sequence, boolean endOfBatch) {
            System.out.println("Event: " + event.getValue() +
                             ", sequence: " + sequence +
                             ", endOfBatch: " + endOfBatch);
        }
    }

    public void disruptorExample() {
        // 配置Disruptor
        int bufferSize = 1024;
        Disruptor<ValueEvent> disruptor = new Disruptor<>(
            new ValueEventFactory(),
            bufferSize,
            Executors.defaultThreadFactory(),
            ProducerType.SINGLE,  // 单生产者
            new BlockingWaitStrategy()  // 阻塞等待策略
        );

        // 设置事件处理器
        disruptor.handleEventsWith(new ValueEventHandler());

        // 启动Disruptor
        RingBuffer<ValueEvent> ringBuffer = disruptor.start();

        // 发布事件
        for (long i = 0; i < 10; i++) {
            long sequence = ringBuffer.next();
            try {
                ValueEvent event = ringBuffer.get(sequence);
                event.setValue(i);
            } finally {
                ringBuffer.publish(sequence);
            }
        }

        // 关闭Disruptor
        disruptor.shutdown();
    }

    // Disruptor的核心特性：
    // 1. 环形缓冲区（Ring Buffer）
    // 2. 无锁设计
    // 3. 批处理事件
    // 4. 多种等待策略

    // 等待策略比较：
    // - BlockingWaitStrategy：CPU使用率低，延迟高
    // - SleepingWaitStrategy：CPU使用率中等，延迟中等
    // - YieldingWaitStrategy：CPU使用率高，延迟低
    // - BusySpinWaitStrategy：CPU使用率极高，延迟极低

    // Disruptor的性能优势：
    // 1. 消除了锁竞争
    // 2. 减少了内存分配
    // 3. 优化了缓存行使用
    // 4. 批量处理事件
}
```

### 6.2 Akka Actor模型

```java
// Akka Actor模型的示例（概念性代码）
public class AkkaActorModel {
    // 1. Actor类
    public static class MyActor extends akka.actor.UntypedActor {
        @Override
        public void onReceive(Object message) throws Exception {
            if (message instanceof String) {
                System.out.println("Received string: " + message);
            } else if (message instanceof Integer) {
                System.out.println("Received integer: " + message);
            } else {
                unhandled(message);
            }
        }
    }

    public void akkaExample() {
        // 1. 创建ActorSystem
        akka.actor.ActorSystem system = akka.actor.ActorSystem.create("MySystem");

        // 2. 创建Actor
        akka.actor.ActorRef myActor = system.actorOf(akka.actor.Props.create(MyActor.class), "myActor");

        // 3. 发送消息
        myActor.tell("Hello", akka.actor.ActorRef.noSender());
        myActor.tell(42, akka.actor.ActorRef.noSender());

        // 4. 关闭系统
        system.terminate();
    }

    // Actor模型的特性：
    // 1. 封装状态
    // 2. 通过消息通信
    // 3. 异步处理
    // 4. 位置透明
    // 5. 容错性

    // Actor模型的优势：
    // 1. 避免了锁竞争
    // 2. 简化了并发编程
    // 3. 提高了可扩展性
    // 4. 增强了容错性
}
```

## 7. 并发编程最佳实践

### 7.1 性能优化策略

```java
public class PerformanceOptimization {
    // 1. 减少锁竞争
    private final Striped<Lock> stripedLocks = Striped.lock(32);
    private final Map<String, Object> data = new ConcurrentHashMap<>();

    public void stripedLockingExample() {
        String key = "user123";
        Lock lock = stripedLocks.get(key);
        lock.lock();
        try {
            // 使用细粒度锁减少竞争
            Object value = data.get(key);
            // 处理数据
        } finally {
            lock.unlock();
        }
    }

    // 2. 使用读写锁
    private final ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
    private volatile String cachedData;

    public String readWithCache() {
        rwLock.readLock().lock();
        try {
            String result = cachedData;
            if (result == null) {
                // 释放读锁，获取写锁
                rwLock.readLock().unlock();
                rwLock.writeLock().lock();
                try {
                    // 双重检查
                    result = cachedData;
                    if (result == null) {
                        result = loadFromDatabase();
                        cachedData = result;
                    }
                    // 降级为读锁
                    rwLock.readLock().lock();
                } finally {
                    rwLock.writeLock().unlock();
                }
            }
            return result;
        } finally {
            rwLock.readLock().unlock();
        }
    }

    private String loadFromDatabase() {
        // 模拟数据库查询
        return "data from database";
    }

    // 3. 使用线程局部变量
    private final ThreadLocal<SimpleDateFormat> dateFormat =
        ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));

    public String formatDate(Date date) {
        return dateFormat.get().format(date);
    }

    // 4. 避免过度同步
    public void avoidOverSynchronization() {
        // 不好的做法：同步整个方法
        public synchronized void badSynchronization() {
            // 只有这部分需要同步
            int result = expensiveCalculation();
            // 同步部分
            sharedVariable++;
        }

        // 好的做法：只同步必要的代码
        public void goodSynchronization() {
            int result = expensiveCalculation();
            synchronized (this) {
                sharedVariable++;
            }
        }
    }

    // 5. 使用不可变对象
    public final class ImmutableData {
        private final int id;
        private final String name;
        private final List<String> items;

        public ImmutableData(int id, String name, List<String> items) {
            this.id = id;
            this.name = name;
            this.items = Collections.unmodifiableList(new ArrayList<>(items));
        }

        // 只提供getter方法
        public int getId() { return id; }
        public String getName() { return name; }
        public List<String> getItems() { return items; }
    }

    private int sharedVariable;
    private int expensiveCalculation() {
        return 0;
    }
}
```

### 7.2 常见问题及解决方案

```java
public class CommonProblems {
    // 1. 死锁问题
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();

    public void deadlockExample() {
        // 可能导致死锁的代码
        Thread thread1 = new Thread(() -> {
            synchronized (lock1) {
                System.out.println("Thread 1 acquired lock1");
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
                synchronized (lock2) {
                    System.out.println("Thread 1 acquired lock2");
                }
            }
        });

        Thread thread2 = new Thread(() -> {
            synchronized (lock2) {
                System.out.println("Thread 2 acquired lock2");
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
                synchronized (lock1) {
                    System.out.println("Thread 2 acquired lock1");
                }
            }
        });

        thread1.start();
        thread2.start();
    }

    // 解决方案：按固定顺序获取锁
    public void safeLocking() {
        Thread thread1 = new Thread(() -> {
            synchronized (lock1) {
                System.out.println("Thread 1 acquired lock1");
                synchronized (lock2) {
                    System.out.println("Thread 1 acquired lock2");
                }
            }
        });

        Thread thread2 = new Thread(() -> {
            synchronized (lock1) {  // 按相同顺序获取锁
                System.out.println("Thread 2 acquired lock1");
                synchronized (lock2) {
                    System.out.println("Thread 2 acquired lock2");
                }
            }
        });

        thread1.start();
        thread2.start();
    }

    // 2. 活锁问题
    private final Lock liveLock1 = new ReentrantLock();
    private final Lock liveLock2 = new ReentrantLock();

    public void livelockExample() {
        Thread thread1 = new Thread(() -> {
            while (true) {
                if (liveLock1.tryLock()) {
                    try {
                        if (liveLock2.tryLock()) {
                            try {
                                System.out.println("Thread 1 working");
                                break;
                            } finally {
                                liveLock2.unlock();
                            }
                        }
                    } finally {
                        liveLock1.unlock();
                    }
                }
                Thread.yield();  // 礼让CPU，可能导致活锁
            }
        });

        Thread thread2 = new Thread(() -> {
            while (true) {
                if (liveLock2.tryLock()) {
                    try {
                        if (liveLock1.tryLock()) {
                            try {
                                System.out.println("Thread 2 working");
                                break;
                            } finally {
                                liveLock1.unlock();
                            }
                        }
                    } finally {
                        liveLock2.unlock();
                    }
                }
                Thread.yield();  // 礼让CPU，可能导致活锁
            }
        });

        thread1.start();
        thread2.start();
    }

    // 解决方案：添加随机延迟
    public void safeLockingWithBackoff() {
        Thread thread1 = new Thread(() -> {
            Random random = new Random();
            while (true) {
                if (liveLock1.tryLock()) {
                    try {
                        if (liveLock2.tryLock()) {
                            try {
                                System.out.println("Thread 1 working");
                                break;
                            } finally {
                                liveLock2.unlock();
                            }
                        }
                    } finally {
                        liveLock1.unlock();
                    }
                }
                try {
                    Thread.sleep(random.nextInt(100));  // 随机延迟
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    break;
                }
            }
        });

        thread1.start();
    }

    // 3. 线程泄漏
    private final ExecutorService executor = Executors.newFixedThreadPool(10);

    public void threadLeakExample() {
        // 不好的做法：没有关闭线程池
        for (int i = 0; i < 100; i++) {
            executor.submit(() -> {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }
        // 没有调用executor.shutdown()
    }

    // 解决方案：正确管理线程池
    public void properThreadPoolManagement() {
        ExecutorService properExecutor = Executors.newFixedThreadPool(10);
        try {
            for (int i = 0; i < 100; i++) {
                properExecutor.submit(() -> {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                    }
                });
            }
        } finally {
            properExecutor.shutdown();
            try {
                if (!properExecutor.awaitTermination(60, TimeUnit.SECONDS)) {
                    properExecutor.shutdownNow();
                }
            } catch (InterruptedException e) {
                properExecutor.shutdownNow();
                Thread.currentThread().interrupt();
            }
        }
    }

    // 4. 竞争条件
    private volatile boolean flag = false;

    public void raceConditionExample() {
        Thread thread1 = new Thread(() -> {
            while (!flag) {
                // 空循环
            }
            System.out.println("Thread 1 detected flag change");
        });

        Thread thread2 = new Thread(() -> {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            flag = true;
            System.out.println("Thread 2 set flag to true");
        });

        thread1.start();
        thread2.start();
    }

    // 解决方案：使用适当的同步机制
    public void safeConditionChecking() {
        final Object lock = new Object();
        Thread thread1 = new Thread(() -> {
            synchronized (lock) {
                while (!flag) {
                    try {
                        lock.wait();  // 等待通知
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                        return;
                    }
                }
                System.out.println("Thread 1 detected flag change");
            }
        });

        Thread thread2 = new Thread(() -> {
            synchronized (lock) {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    return;
                }
                flag = true;
                lock.notifyAll();  // 通知等待的线程
                System.out.println("Thread 2 set flag to true");
            }
        });

        thread1.start();
        thread2.start();
    }
}
```

## 8. 并发编程调试与测试

### 8.1 并发调试技巧

```java
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;

public class ConcurrentDebugging {
    // 1. 使用日志记录并发操作
    private final AtomicInteger counter = new AtomicInteger(0);

    public void concurrentOperationWithLogging() {
        ExecutorService executor = Executors.newFixedThreadPool(5);

        for (int i = 0; i < 100; i++) {
            final int taskId = i;
            executor.submit(() -> {
                System.out.println("Task " + taskId + " started by " +
                                 Thread.currentThread().getName());

                int currentValue = counter.incrementAndGet();
                System.out.println("Task " + taskId + " counter: " + currentValue +
                                 " by " + Thread.currentThread().getName());

                System.out.println("Task " + taskId + " completed by " +
                                 Thread.currentThread().getName());
            });
        }

        executor.shutdown();
    }

    // 2. 使用线程安全的集合进行调试
    private final ConcurrentLinkedQueue<String> debugLog = new ConcurrentLinkedQueue<>();

    public void debugWithConcurrentCollection() {
        ExecutorService executor = Executors.newFixedThreadPool(5);

        for (int i = 0; i < 10; i++) {
            final int taskId = i;
            executor.submit(() -> {
                debugLog.add("Task " + taskId + " started");

                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }

                debugLog.add("Task " + taskId + " completed");
            });
        }

        executor.shutdown();

        try {
            executor.awaitTermination(5, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }

        // 输出调试日志
        while (!debugLog.isEmpty()) {
            System.out.println(debugLog.poll());
        }
    }

    // 3. 使用CountDownLatch进行同步调试
    public void debugWithCountDownLatch() {
        final int threadCount = 3;
        final CountDownLatch startLatch = new CountDownLatch(1);
        final CountDownLatch endLatch = new CountDownLatch(threadCount);

        for (int i = 0; i < threadCount; i++) {
            final int threadId = i;
            new Thread(() -> {
                try {
                    System.out.println("Thread " + threadId + " ready");
                    startLatch.await();  // 等待开始信号

                    System.out.println("Thread " + threadId + " started");
                    Thread.sleep(1000);
                    System.out.println("Thread " + threadId + " completed");

                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    endLatch.countDown();
                }
            }).start();
        }

        // 等待所有线程准备就绪
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }

        System.out.println("Starting all threads...");
        startLatch.countDown();  // 发送开始信号

        try {
            endLatch.await();  // 等待所有线程完成
            System.out.println("All threads completed");
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }

    // 4. 使用JMX进行监控
    public void jmxMonitoring() {
        // 获取MBean服务器
        MBeanServer mbs = ManagementFactory.getPlatformMBeanServer();

        try {
            // 获取线程MBean
            ThreadMXBean threadBean = ManagementFactory.getThreadMXBean();

            // 获取线程信息
            long[] threadIds = threadBean.getAllThreadIds();
            for (long threadId : threadIds) {
                ThreadInfo info = threadBean.getThreadInfo(threadId);
                System.out.println("Thread ID: " + threadId +
                                 ", Name: " + info.getThreadName() +
                                 ", State: " + info.getThreadState());
            }

            // 获取内存MBean
            MemoryMXBean memoryBean = ManagementFactory.getMemoryMXBean();
            MemoryUsage heapUsage = memoryBean.getHeapMemoryUsage();
            System.out.println("Heap Memory Used: " + heapUsage.getUsed() + " bytes");
            System.out.println("Heap Memory Max: " + heapUsage.getMax() + " bytes");

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    // 5. 使用VisualVM进行性能分析
    public void visualVMProfiling() {
        // 提示用户使用VisualVM进行分析
        System.out.println("To profile this application:");
        System.out.println("1. Run 'jvisualvm' from command line");
        System.out.println("2. Connect to the running Java process");
        System.out.println("3. Monitor threads, memory, and CPU usage");
        System.out.println("4. Take thread dumps and heap dumps");

        // 模拟一些并发操作以便分析
        ExecutorService executor = Executors.newFixedThreadPool(10);
        for (int i = 0; i < 100; i++) {
            executor.submit(() -> {
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }

        executor.shutdown();
        try {
            executor.awaitTermination(10, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```

### 8.2 并发测试策略

```java
import org.junit.Test;
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;

public class ConcurrentTesting {
    // 1. 基本并发测试
    @Test
    public void basicConcurrentTest() throws InterruptedException {
        final AtomicInteger counter = new AtomicInteger(0);
        final int threadCount = 10;
        final int incrementsPerThread = 1000;
        ExecutorService executor = Executors.newFixedThreadPool(threadCount);

        // 创建并提交任务
        for (int i = 0; i < threadCount; i++) {
            executor.submit(() -> {
                for (int j = 0; j < incrementsPerThread; j++) {
                    counter.incrementAndGet();
                }
            });
        }

        executor.shutdown();
        executor.awaitTermination(10, TimeUnit.SECONDS);

        // 验证结果
        int expected = threadCount * incrementsPerThread;
        int actual = counter.get();
        System.out.println("Expected: " + expected + ", Actual: " + actual);
        assert expected == actual : "Counter value mismatch";
    }

    // 2. 压力测试
    @Test
    public void stressTest() throws InterruptedException {
        final int threadCount = 100;
        final int iterations = 10000;
        final ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
        ExecutorService executor = Executors.newFixedThreadPool(threadCount);
        CountDownLatch latch = new CountDownLatch(threadCount);

        // 记录开始时间
        long startTime = System.currentTimeMillis();

        // 提交任务
        for (int i = 0; i < threadCount; i++) {
            final int threadId = i;
            executor.submit(() -> {
                try {
                    for (int j = 0; j < iterations; j++) {
                        String key = "key-" + threadId + "-" + j;
                        map.put(key, j);
                    }
                } finally {
                    latch.countDown();
                }
            });
        }

        // 等待所有任务完成
        latch.await();
        long endTime = System.currentTimeMillis();

        // 输出性能指标
        long duration = endTime - startTime;
        double throughput = (double) (threadCount * iterations) / duration * 1000;
        System.out.println("Stress test completed in " + duration + " ms");
        System.out.println("Throughput: " + throughput + " operations/second");
        System.out.println("Map size: " + map.size());

        executor.shutdown();
    }

    // 3. 死锁测试
    @Test(timeout = 5000)
    public void deadlockTest() throws InterruptedException {
        final Object lock1 = new Object();
        final Object lock2 = new Object();
        final CountDownLatch latch = new CountDownLatch(2);

        Thread thread1 = new Thread(() -> {
            synchronized (lock1) {
                try {
                    latch.countDown();
                    latch.await();  // 等待thread2也准备好
                    synchronized (lock2) {
                        System.out.println("Thread 1 acquired both locks");
                    }
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        });

        Thread thread2 = new Thread(() -> {
            synchronized (lock2) {
                try {
                    latch.countDown();
                    latch.await();  // 等待thread1也准备好
                    synchronized (lock1) {
                        System.out.println("Thread 2 acquired both locks");
                    }
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        });

        thread1.start();
        thread2.start();

        // 等待死锁发生或超时
        thread1.join();
        thread2.join();
    }

    // 4. 性能基准测试
    @Test
    public void performanceBenchmark() {
        final int iterations = 1000000;

        // 测试synchronized性能
        long syncStart = System.nanoTime();
        testSynchronized(iterations);
        long syncEnd = System.nanoTime();
        double syncDuration = (syncEnd - syncStart) / 1_000_000.0;

        // 测试AtomicInteger性能
        long atomicStart = System.nanoTime();
        testAtomicInteger(iterations);
        long atomicEnd = System.nanoTime();
        double atomicDuration = (atomicEnd - atomicStart) / 1_000_000.0;

        // 测试LongAdder性能
        long adderStart = System.nanoTime();
        testLongAdder(iterations);
        long adderEnd = System.nanoTime();
        double adderDuration = (adderEnd - adderStart) / 1_000_000.0;

        // 输出性能比较结果
        System.out.println("Synchronized duration: " + syncDuration + " ms");
        System.out.println("AtomicInteger duration: " + atomicDuration + " ms");
        System.out.println("LongAdder duration: " + adderDuration + " ms");
        System.out.println("Atomic improvement: " + (syncDuration / atomicDuration) + "x");
        System.out.println("LongAdder improvement: " + (syncDuration / adderDuration) + "x");
    }

    private void testSynchronized(int iterations) {
        final Object lock = new Object();
        final AtomicInteger counter = new AtomicInteger(0);

        for (int i = 0; i < iterations; i++) {
            synchronized (lock) {
                counter.incrementAndGet();
            }
        }
    }

    private void testAtomicInteger(int iterations) {
        final AtomicInteger counter = new AtomicInteger(0);

        for (int i = 0; i < iterations; i++) {
            counter.incrementAndGet();
        }
    }

    private void testLongAdder(int iterations) {
        final LongAdder adder = new LongAdder();

        for (int i = 0; i < iterations; i++) {
            adder.increment();
        }
    }

    // 5. 使用JMH进行专业基准测试
    /*
    @Benchmark
    @BenchmarkMode(Mode.AverageTime)
    @OutputTimeUnit(TimeUnit.NANOSECONDS)
    public void jmhBenchmark() {
        // 使用JMH框架进行专业的性能基准测试
        // 需要单独的JMH测试类
    }
    */
}
```

## 9. 未来发展趋势

### 9.1 虚拟线程（Virtual Threads）

```java
public class VirtualThreads {
    public void virtualThreadExample() {
        // Java 19引入的虚拟线程
        // 虚拟线程是轻量级线程，由JVM管理

        // 创建虚拟线程
        Thread virtualThread = Thread.ofVirtual().start(() -> {
            System.out.println("Running in virtual thread: " + Thread.currentThread());
        });

        // 使用虚拟线程池
        ExecutorService virtualExecutor = Executors.newVirtualThreadPerTaskExecutor();

        for (int i = 0; i < 1000; i++) {
            final int taskId = i;
            virtualExecutor.submit(() -> {
                System.out.println("Task " + taskId + " in " + Thread.currentThread());
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }

        virtualExecutor.shutdown();
        try {
            virtualExecutor.awaitTermination(10, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }

    // 虚拟线程的优势：
    // 1. 轻量级，可以创建大量虚拟线程
    // 2. 适合I/O密集型任务
    // 3. 减少内存消耗
    // 4. 简化并发编程

    // 虚拟线程的使用场景：
    // 1. 大量并发连接的服务器
    // 2. 微服务架构
    // 3. 异步I/O操作
    // 4. 事件驱动应用
}
```

### 9.2 结构化并发

```java
import java.util.concurrent.StructuredTaskScope;
import java.util.concurrent.StructuredTaskScope.Subtask;

public class StructuredConcurrency {
    public void structuredConcurrencyExample() {
        try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
            // 启动多个子任务
            Subtask<String> userTask = scope.fork(() -> fetchUserData());
            Subtask<String> orderTask = scope.fork(() -> fetchOrderData());

            // 等待所有任务完成或失败
            scope.join();

            // 获取结果
            String user = userTask.get();
            String order = orderTask.get();

            System.out.println("User: " + user + ", Order: " + order);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }

    private String fetchUserData() {
        try {
            Thread.sleep(500);
            return "User Data";
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            throw new RuntimeException("Interrupted");
        }
    }

    private String fetchOrderData() {
        try {
            Thread.sleep(300);
            return "Order Data";
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            throw new RuntimeException("Interrupted");
        }
    }

    // 结构化并发的特性：
    // 1. 任务生命周期管理
    // 2. 错误传播
    // 3. 任务取消
    // 4. 资源管理

    // 结构化并发的优势：
    // 1. 更好的错误处理
    // 2. 避免任务泄漏
    // 3. 代码更清晰
    // 4. 更容易调试
}
```

## 10. 最佳实践总结

### 10.1 并发编程原则

```java
public class ConcurrencyPrinciples {
    // 1. 最小化共享状态
    private final int threadLocalValue = 42;

    // 2. 使用不可变对象
    public final class ImmutableConfig {
        private final String apiUrl;
        private final int timeout;

        public ImmutableConfig(String apiUrl, int timeout) {
            this.apiUrl = apiUrl;
            this.timeout = timeout;
        }

        // 只提供getter方法
        public String getApiUrl() { return apiUrl; }
        public int getTimeout() { return timeout; }
    }

    // 3. 选择合适的同步机制
    public void chooseAppropriateSynchronization() {
        // 低竞争：synchronized
        // 高竞争读多写少：ReadWriteLock
        // 简单计数：Atomic类
        // 复杂操作：Lock
        // 高性能需求：无锁数据结构
    }

    // 4. 避免过早优化
    public void avoidPrematureOptimization() {
        // 先写正确的代码
        // 再进行性能测试
        // 最后进行优化
    }

    // 5. 正确处理异常
    public void properExceptionHandling() {
        Lock lock = new ReentrantLock();
        try {
            lock.lock();
            // 业务逻辑
        } finally {
            lock.unlock();  // 确保锁被释放
        }
    }

    // 6. 使用线程池
    public void useThreadPool() {
        ExecutorService executor = Executors.newFixedThreadPool(
            Runtime.getRuntime().availableProcessors()
        );

        try {
            executor.submit(() -> {
                // 任务逻辑
            });
        } finally {
            executor.shutdown();  // 确保线程池被关闭
        }
    }
}
```

### 10.2 性能调优建议

```java
public class PerformanceTuning {
    // 1. 线程池大小调优
    public void optimalThreadPoolSize() {
        // CPU密集型任务：线程数 = CPU核心数
        int cpuBoundThreads = Runtime.getRuntime().availableProcessors();

        // I/O密集型任务：线程数 = CPU核心数 * (1 + 平均等待时间/平均服务时间)
        int ioBoundThreads = cpuBoundThreads * 2;

        // 混合型任务：需要根据实际情况调整
        int mixedThreads = cpuBoundThreads * 4;
    }

    // 2. 批量处理
    public void batchProcessing() {
        List<Task> tasks = getTasks();
        ExecutorService executor = Executors.newFixedThreadPool(10);

        // 批量提交任务
        List<Future<Result>> futures = new ArrayList<>();
        for (Task task : tasks) {
            futures.add(executor.submit(() -> processTask(task)));
        }

        // 批量处理结果
        for (Future<Result> future : futures) {
            try {
                Result result = future.get();
                // 处理结果
            } catch (InterruptedException | ExecutionException e) {
                e.printStackTrace();
            }
        }

        executor.shutdown();
    }

    // 3. 缓存优化
    private final Map<String, Object> cache = new ConcurrentHashMap<>();

    public Object getCachedResult(String key) {
        return cache.computeIfAbsent(key, k -> {
            // 计算或获取结果
            return expensiveOperation(k);
        });
    }

    private Object expensiveOperation(String key) {
        // 模拟昂贵操作
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        return "Result for " + key;
    }

    private List<Task> getTasks() {
        return new ArrayList<>();
    }

    private Result processTask(Task task) {
        return new Result();
    }

    private static class Task {}
    private static class Result {}
}
```

## 结语

并发编程是Java开发中的核心技能，从基础的synchronized到高性能的Disruptor框架，Java提供了丰富的并发工具和机制。通过本文的学习，你应该掌握了：

1. Java并发编程的基础概念和原理
2. synchronized和volatile的深入理解
3. java.util.concurrent包的核心组件
4. 高级并发模式和框架
5. 性能优化和调试技巧
6. 最佳实践和常见问题解决方案

记住，并发编程是一门艺术，需要理论知识和实践经验相结合。**优秀的并发代码不仅要正确，还要高效、可维护。** 在实际项目中，要根据具体场景选择合适的并发策略，平衡性能、复杂度和可维护性。

随着Java的发展，虚拟线程、结构化并发等新特性将使并发编程变得更加简单和高效。保持学习，跟上技术发展的步伐，才能在并发编程的道路上不断进步。

---

*这篇文章深入探讨了Java并发编程的各个方面，从基础概念到高级技术，从理论原理到实践经验。通过丰富的代码示例和实际案例分析，帮助读者全面理解Java并发编程的精髓。浅者可以掌握基本的并发编程技巧，深者能够深入理解底层原理和性能优化策略。*