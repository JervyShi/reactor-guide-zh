# 什么是Reactor ?

你想来看一看什么是Reactor。可能你会用你喜欢的搜索引擎来搜索这些关键字，比如 Reactive，Spring+Reactive，Asynchronous+Java或者仅仅是“What the heck is Reactor？”。简而言之，Reactor是一个轻量级的JVM基础库，它可以帮助我们构建的服务或应用高效而异步的传递消息。


**_高效_ 的含义是什么?**

* 一个消息从A传递到B产生的内存很小或者完全没有。
* 当消费者的消费速度低于生产者生产速度而产生溢出时，必须尽快处理
* 尽可能提供无阻塞的异步流

据以往的经验看（主要通过 #rage 和 #drunk 的tweets），我们知道异步编程是困难的，特别是当一个平台提供大量的选项，比如JVM。Reactor的目标是在大部分场景下实现真正的无阻塞，并且提供一组比原生JDK中`java.util.concurrent`更高效的API。Reactor也提供替代方案（不鼓励使用）：

* 阻塞等待： 例如 `Future.get()`

* 不安全的数据访问： 例如 `ReentrantLock.lock()`

* 异常块： 例如 `try…catch…finally`

* 同步阻塞： 例如 `synchronized{ }`

* Wrapper分配 (GC 压力)： 例如 `new Wrapper<T>(event)`

消息传递以非阻塞的方式进行，在消息传递量极大时就变的更加关键（10k msg/s, 100k msg/s 1M…）。这里有背后的一些理论（[Amdahl’s Law](https://en.wikipedia.org/wiki/Amdahl%27s_law)），但我们容易对理论感到厌倦，容易分心，所以我们呼吁先了解一些常识。

让我们假设你使用一个纯正的Executor方法：

```
private ExecutorService  threadPool = Executors.newFixedThreadPool(8);

final List<T> batches = new ArrayList<T>();

Callable<T> t = new Callable<T>() { (1)

    public T run() {
        synchronized(batches) { (2)
            T result = callDatabase(msg); (3)
            batches.add(result);
            return result;
        }
    }
};

Future<T> f = threadPool.submit(t); (4)
T result = f.get() (5)
```

1. 分配回调方法--可能会导致gc压力。
2. Synchronization将强制对每个线程实施停止检查。
3. 可能存在消费者消费能力低于生产者生产能力的隐患。
4. 使用线程池将task传递到目标线程--肯定会通过FutureTask给gc造成压力。
5. 阻塞直至`callDatabase()`响应。

在这个简单的例子中，很容易看出为什么扩展性是非常有限的：

* 不断分配的对象可能导致GC暂停，特别是当有一些任务耗时过长。

 * 每一次GC暂停都将在全局范围内降低性能。

* 队列默认是无界的。由于对数据库的调用，任务将产生堆积。

 * A backlog is not really a Memory Leak but the side effects are just as nasty: more objects to scan during GC pauses; risk of losing important bits of data; etc…

 * Classic Linked Queues generate memory pressure by allocating Nodes. Lots of them.

* A vicious cycle kicks-in when blocking replies are used.

 * Blocking replies will cause producer slow-down. In practice, the flow becomes basically synchronous since we have to wait for each reply before submitting more tasks.

 * Any Exception thrown during the conversation with the datastore will be passed in an uncontrolled fashion to the producer, negating any fault-tolerance normally available by segregating work around a Thread boundary.

Being fully and truly non-blocking is hard to achieve—especially in a world of distributed systems which have fashionable monikers like Micro-Service Architectures. Reactor, however, makes few compromises and tries to leverage the best patterns available so the developer doesn’t have to feel like they’re writing a mathematical thesis rather than an asynchronous nanoservice.

Nothing travels faster than light (besides gossip and viral cat videos) and latency is a real-world concern every system has to deal with at some point. To that end:

**Reactor offers a framework that helps you mitigate nasty latency-induced side-effects in your application and do it with minimal overhead by:**

* Leveraging some smart structures, we traded-off the allocation issue at runtime with pre-allocation at startup-time;

* Main message-passing structures come bounded so we don’t pile up tasks infinitely;

* Using popular patterns such as Reactive and Event-Driven Architectures, we offer non-blocking end-to-end flows including replies;

* Implementing the new Reactive Streams Standard, to make bounded structures efficient by not requesting more than their current capacity;

* Applied these concepts to IPC and provide non-blocking IO drivers that understand flow-control;

* Expose a Functional API to help developers organize their code in a side-effect free way, which helps you determine where you are thread-safe and fault-tolerant.
