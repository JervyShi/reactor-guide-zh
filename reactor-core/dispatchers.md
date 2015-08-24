# Dispatcher

从Reactor 1时就已经有Dispatcher了。Dispatcher通常抽象消息传递的方法，类似Java中的Executer。事实上，Dispathcer继承自Executor！

Dispatcher提供一种强类型的方式来传递数据和错误给同步或异步执行的消费者。我们在这方面克服了一个传统Executor首要面对的问题：错误隔离。实际上，Error Consumer将会被调用来代替传统的分配资源中断。如果在当前Environment中没有任何consumer被分配，将使用默认的errorJournalConsumer来处理异常。

异步的Dispatcher带来的第二个独立特性是允许使用尾递归策略来重复调度。尾部递归的应用场景是分发器发现Dispatcher的classLoader已经分配到正在运行的线程，这时，当当前消费者返回时将要执行的task放入到队列中。

使用同步和多线程dispather，例如下列Groovy Spock测试：
Using a synchronous and a multi-threaded dispatcher like in this Groovy Spock test:

```
import reactor.core.dispatch.*

//...

given:
  def sameThread = new SynchronousDispatcher()
  def diffThread = new ThreadPoolExecutorDispatcher(1, 128)
  def currentThread = Thread.currentThread()
  Thread taskThread = null

  def consumer = { ev ->
    taskThread = Thread.currentThread()
  }

  def errorConsumer = { error ->
    error.printStackTrace()
  }

when: "a task is submitted"
  sameThread.dispatch('test', consumer, errorConsumer)

then: "the task thread should be the current thread"
  currentThread == taskThread

when: "a task is submitted to the thread pool dispatcher"
  def latch = new CountDownLatch(1)
  diffThread.dispatch('test', { ev -> consumer(ev); latch.countDown() }, errorConsumer)

  latch.await(5, TimeUnit.SECONDS) // Wait for task to execute

then: "the task thread should be different when the current thread"
  taskThread != currentThread
```

> 就像Executor一样，他们将缺少一个我们将在2.x release版本线中增加的特性：Reactive Stream协议。这时在Reactor中仅有几个未完成事项中的一个未完成事项——没有将Reactive stream标准直接绑定到Reactor中。然后，你可以在Stream章节部分找到快速结合Reactor Stream的方法。

Table 3. Dispatcher家族的介绍

Dispatcher|From Environment|Description|Strengths|Weaknesses
----------|----------------|-----------|---------|----------
RingBuffer|sharedDispatcher()|An LMAX Disruptor RingBuffer based Dispatcher.|Small latency peaks tolerated Fastest Async Dispatcher, 10-15M+ dispatch/sec on commodity hardware Support ordering|'Spin' Loop when getting the next slot on full capcity Single Threaded, no concurrent dispatch
Mpsc|sharedDispatcher() if Unsafe not available|Alternative optimized message-passing structure.|Latency peaks tolerated 5-10M+ dispatch/sec on commodity hardware Support ordering|Unbounded and possibly using as much available heap memory as possible \n Single Threaded, no concurrent dispatch
WorkQueue|workDispatcher()|An LMAX Disruptor RingBuffer based Dispatcher.|Latency Peak tolerated for a limited time \n Fastest Multi-Threaded Dispatcher, 5-10M+ dispatch/sec on commodity hardware|'Spin' Loop when getting the next slot on full capcity \n Concurrent dispatch \n Doesn’t support ordering
Synchronous|dispatcher("sync") or SynchronousDispatcher. INSTANCE|Runs on the current thread.|Upstream and Consumer executions are colocated \n Useful for Test support \n Support ordering if the reentrant dispatch is on the current thread|No Tail Recursion support \n Blocking
TailRecurse|tailRecurse() or TailRecurse Dispatcher. INSTANCE|Synchronous Reentrant Dispatcher that enqueue dispatches when currently dispatching.|Upstream and Consumer executions are colocated \n Reduce execution stack, greatly expanded by functional call chains|Unbounded Tail Recurse depth \n Blocking \n Support ordering (Thread Stealing)
ThreadPoolExecutor|newDispatcher(int, int, DispatcherType. THREAD_POOL_EXECUTOR)|Use underlying ThreadPoolExecutor message-passing|Multi-Threaded \n Blocking Consumers, permanent latency tolerated \n 1-5M+ dispatch/sec on commodity hardware|Concurrent run on a given consumer executed twice or more \n Unbounded by default \n Doesn’t support ordering
Traceable Delegating|N/A|Decorate an existing dispatcher with TRACE level logs. |Dispatch tapping \n Runs slower than the delegated dispatcher alone|Log overhead (runtime, disk)

![Figure 6. RingBufferDispatcher at a given time T](http://projectreactor.io/docs/reference/images/rbd2.png)
**Figure 6. RingBufferDispatcher at a given time T**


