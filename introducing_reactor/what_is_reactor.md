# What is Reactor ?

So you came to have a look at Reactor. Maybe you typed some keywords into your favorite search engine like Reactive, Spring+Reactive, Asynchronous+Java or just What the heck is Reactor?. In a nutshell Reactor is a lightweight, foundational library for the JVM that helps your service or application to efficiently and asynchronously pass messages.


**What do you mean by "efficiently"?**

* Little to no memory garbage created just to pass a message from A to B.
* Handle overflow when consumers are slower at processing messages than the producer is at producing them.
* Provide for asynchronous flow--without blocking—if at all possible.

From empirical studies (mostly #rage and #drunk tweets), we know that asynchronous programming is hard—especially when a platform like the JVM offers so many options. Reactor aims to be truly non-blocking for a majority of use cases and we offer an API that is measurably more efficient than relying on low-level primitives from the JDK’s java.util.concurrent library. Reactor provides alternatives to (and discourages the use of):

* Blocking wait : e.g. Future.get()

* Unsafe data access : e.g. ReentrantLock.lock()

* Exception Bubbles : e.g. try…catch…finally

* Synchronization blocks : e.g. synchronized{ }

* Wrapper Allocation (GC Pressure) : e.g. new Wrapper<T>(event)

Being non-blocking matters—especially when scaling message-passing becomes critical (10k msg/s, 100k msg/s 1M…). There is some theory behind this (see Amdahl’s Law), but we get bored and distracted easily, so let’s first appeal to common sense.

Let’s say you use a pure Executor approach:

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

1. Allocate Callable—might lead to GC pressure.
2. Synchronization will force stop-and-check for every thread.
3. Potentially consumes slower than producer produces.
4. Use a ThreadPool to pass the task to the target thread—definitely produces GC pressure via FutureTask.
5. Block until callDatabase() replies.

In this simple example, it’s easy to point out why scale-up is very limited:

* Allocating objects might cause GC pauses, especially if the tasks stick around too long.

 * Every GC Pause will degrade performance globally.

* A Queue is unbounded by default. Because of the database call, tasks will pile up.

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
