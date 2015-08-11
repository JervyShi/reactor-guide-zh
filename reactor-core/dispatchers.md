# Dispatchers
Dispatchers are there since Reactor 1, they abstract away the mean of message-passing in a common contract similar to the Java Executor. In fact they do extend Executor!

The Dispatcher contract offers a strongly typed way to pass a signal with its Data and Error Consumers executed (a)synchronously. This way we fix a first issue faced by classic Executors, the error isolation. In effect instead of interrupting the assigned resource, the Error Consumer will be invoked. If none has been provided it will try to find an existing Environment and use its assigned errorJournalConsumer.

A second unique feature offered by the asynchronous Dispatcher is to allow for reentrant dispatching by using a Tail Recurse strategy. Tail Recursion is used when dispatch detects the dispatcher classLoader has been assigned to the running thread and if so, enqueue the task to be executed when the current consumer returns.

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

> Like the Executor they will miss a feature that we will add along the 2.x release line: Reactive Streams protocol. They are ones of the few leftovers in Reactor that are not directly tied to the Reactive Streams standard directly. However, they can be combined with the Reactor Stream to quickly fix that as we will explore in the Stream Section. Essentially that means a user can directly hit them until they eventually and temporarely block since the capacity might be bounded by most Dispatcher implementations.

Table 3. An introduction to the Dispatcher family

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


