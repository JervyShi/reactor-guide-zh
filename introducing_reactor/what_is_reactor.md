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
 * 积压不是真正意义上的内存泄漏，但是其副作用一样讨厌：GC暂停时需要对更多对象进行扫描；丢失数据重要字节的风险；等等...
 * 经典链表队列分配节点时产生内存压力。数不胜数。
* 使用阻塞方式应答请求时发生恶性循环。
 * 阻塞方式应答请求导致生产者效率缓慢。实际上，因为需要提交更多任务时等待响应，流程变成了基本的同步方式。
 * 同数据存储的通信异常将以不友好的形式传递到生产者，通过线程边界来分离工作，这使容错的协商变的比较容易。

完全的、真正的非阻塞比较难以实现--特别是在拥有众多时髦名称的分布式系统世界中如微服务架构。然而，Reactor却没有妥协，它试图利用可用的最佳模式来使开发者不必觉得像是在写一个数学论文而仅仅是一个微服务(Nanoservice)。

没有什么传播速度比光更快（除了八卦和病毒猫的视频），在某些情况下，延迟是现实世界中每个系统都必须关注的。为此：

**Reactor提供一个框架，帮助你在你的应用中使用最小开销来解决延迟带来的副作用：**

* 使用一些灵活的结构，我们通过在启动时预分配空间来避免分配问题；
* 主要的消息传递结构有界，因而不会导致任务无限积压；
* 利用流行的模式例如Reactive和事件驱动架构，我们提供一个包含应答的非阻塞的、端对端的流；
* 实现了最新的[Reactive流](http://projectreactor.io/docs/reference/#reactivestreams)标准，通过不发送多于当前容量的请求来使受限的结构更有效率；
* 使用这些概念到进程间通信，提供了理解控制流的非阻塞IO驱动；
* 暴露函数式API来帮助开发者通过无副作用的方式来组织代码，也帮助你确定在什么场景下你是线程安全和具有容错性的。
