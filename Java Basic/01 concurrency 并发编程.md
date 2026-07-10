> ⭐ 标记 = 高频考点 | ⭐⭐ = 2026 必考/重点

---

# PART 1: 并发编程 (Concurrency) ⭐ PNC 重点

> Ivan 说 "concurrency is something we use heavily over here" —— 这是 PNC 这类岗位的核心,必须吃透。

## 1.1 什么是 Race Condition (竞态条件) ⭐ 被问到

**中文:** 当多个线程并发地读写同一个共享变量,而最终结果取决于线程执行的时序或顺序时,就发生了竞态条件。问题的本质不是"多线程",而是"没有正确同步对共享状态的访问"。

**English answer:**
"A race condition occurs when multiple threads concurrently read and write the same shared variable, and the final outcome depends on the unpredictable timing or order of thread execution. The core issue is not multithreading itself — it's the lack of proper synchronization when accessing shared mutable state."

**经典例子 — i++ 不是原子操作:**
```java
// i++ 实际是三步:read i → i+1 → write i
// 两个线程同时执行,可能都读到 i=5,都写回 6,丢失一次自增
private int count = 0;
public void increment() { count++; }  // NOT thread-safe
```

## 1.2 防止 Race Condition 的四种方法 ⭐ 被问到(卡住了)

> Ivan 明确说有四种,而且要求"besides locking"(除了锁之外)。这是标准答案:

**1. Synchronized / Locking (加锁同步)**
```java
public synchronized void increment() { count++; }
// 或者显式锁
ReentrantLock lock = new ReentrantLock();
lock.lock(); try { count++; } finally { lock.unlock(); }
```
> 中文:最直接的方式,保证同一时刻只有一个线程访问。缺点是有性能开销和死锁风险。

**2. Compare-and-Swap / CAS (无锁原子操作)** —— 你答到了这个方向
```java
AtomicInteger count = new AtomicInteger(0);
count.incrementAndGet();  // 底层用 CPU 的 CAS 指令,无锁
```
> 中文:利用 CPU 底层指令,只有当前值等于预期值时才写入新值。`java.util.concurrent.atomic` 包里的 AtomicInteger、AtomicLong 都是这个原理。无锁,性能好。
> **English:** "CAS uses CPU-level instructions to atomically update a value only if it still matches the expected value — this is how AtomicInteger works, lock-free."

**3. Immutability (不可变对象)**
```java
public final class Money {
    private final BigDecimal amount;  // final,创建后不可改
    public Money(BigDecimal amount) { this.amount = amount; }
}
```
> 中文:如果数据不可变,那么无论多少线程访问都不会被破坏,天然线程安全。String 就是典型的不可变类。
> **English:** "If the data is immutable, it doesn't matter what threads do with it — it cannot be corrupted."

**4. No Sharing / Thread Confinement (不共享数据)**
```java
ThreadLocal<SimpleDateFormat> formatter =
    ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));
// 每个线程有自己的副本,互不干扰
```
> 中文:干脆不共享 —— 每个线程持有自己的数据副本(ThreadLocal),没有共享就没有竞态。
> **English:** "Don't share the data at all — give each thread its own copy via ThreadLocal."

> 💡 **面试记忆口诀:** 锁(synchronized)、无锁(CAS)、不可变(immutable)、不共享(ThreadLocal)。

## 1.3 CompletableFuture vs ExecutorService ⭐ 被问到(卡住了)

> Ivan 说最简单的答法:

**ExecutorService(执行器服务):**
> 中文:它是一个建立在**线程池**之上的"任务提交入口"。你把任务(Runnable/Callable)提交给它,它负责调度线程去执行。
> **English:** "An ExecutorService is essentially a task submission point on top of a thread pool. You submit tasks and it manages the threads that run them."

```java
ExecutorService executor = Executors.newFixedThreadPool(10);
Future<String> future = executor.submit(() -> "result");
String result = future.get();  // 阻塞等待结果
```

**CompletableFuture(可组合的future):**
> 中文:它代表一个**正在线程池里运行的异步任务的结果**,而且可以链式组合、非阻塞回调。
> **English:** "A CompletableFuture is a representation of something running inside a pool — but it supports non-blocking callback chaining."

```java
CompletableFuture.supplyAsync(() -> fetchData())      // 异步执行
    .thenApply(data -> transform(data))                // 链式转换,非阻塞
    .thenAccept(result -> save(result))                // 消费结果
    .exceptionally(ex -> { log(ex); return null; });   // 异常处理
```

**核心区别:**
| | ExecutorService | CompletableFuture |
|---|---|---|
| 本质 | 线程池 + 任务提交 | 异步任务的结果表示 |
| 获取结果 | `future.get()` 阻塞 | 回调链,非阻塞 |
| 组合任务 | 困难 | `thenApply/thenCompose/thenCombine` 轻松组合 |

> 💡 **一句话:** ExecutorService 是"跑任务的池子",CompletableFuture 是"池子里那个任务的结果句柄,还能串起来"。

## 1.4 线程池核心参数 (ThreadPoolExecutor)

```java
new ThreadPoolExecutor(
    corePoolSize,     // 核心线程数(常驻)
    maximumPoolSize,  // 最大线程数
    keepAliveTime,    // 空闲线程存活时间
    unit,             // 时间单位
    workQueue,        // 任务队列(如 LinkedBlockingQueue)
    threadFactory,    // 线程工厂
    handler           // 拒绝策略(队列满 + 线程满时)
);
```
> 中文:任务来了先用核心线程,核心满了进队列,队列满了才扩到 max,再满就触发拒绝策略。这个执行顺序面试常考。

**四种拒绝策略:**
- `AbortPolicy` — 抛异常(默认)
- `CallerRunsPolicy` — 调用者线程自己跑
- `DiscardPolicy` — 静默丢弃
- `DiscardOldestPolicy` — 丢弃队列最老的

## 1.5 volatile 关键字

> 中文:volatile 保证**可见性**和**有序性**,但不保证**原子性**。
> - 可见性:一个线程改了 volatile 变量,其他线程立刻看得到(直接读写主内存,不走 CPU 缓存)。
> - 有序性:禁止指令重排序。
> - 不保证原子性:`volatile int i; i++` 仍然不是线程安全的(因为 i++ 是三步)。

```java
private volatile boolean running = true;  // 典型用法:状态标志位
public void stop() { running = false; }    // 其他线程立刻看到
```

## 1.6 synchronized vs ReentrantLock

| | synchronized | ReentrantLock |
|---|---|---|
| 类型 | 关键字(JVM 层面) | 类(API 层面) |
| 释放锁 | 自动(出了代码块) | 手动(必须 finally 里 unlock) |
| 公平锁 | 不支持 | 支持(构造时传 true) |
| 可中断 | 不可 | 可(lockInterruptibly) |
| 尝试获取 | 不可 | 可(tryLock) |

## 1.7 死锁 (Deadlock) 四个必要条件

> 中文:互斥、持有并等待、不可剥夺、循环等待。破坏任意一个就能避免死锁。最常用的是"按固定顺序加锁"来破坏循环等待。

---