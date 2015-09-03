# RingBufferProcessor
Reactor的`RingBufferProcessor`组件本质上是一个改造过的适应Reactive Streams API的[Disruptor RingBuffer](https://github.com/LMAX-Exchange/disruptor)。设计目的是尽可能和原生效率一样高。使用场景：你需要分发任务到其他线程，并使用极少的资源和极高的吞吐量而且可以在你的工作流中管理背压。

> 我使用`RingBufferProcessor`来计算异步的远程调用各种输出：AMQP，SSD存储和内存存储，可变的延迟几乎全部被`Processor`消灭，我们每秒百万消息的数据源从未阻塞。

> -- 开心的Reactor使用者

> `RingBufferProcessor`的使用案例

![图7. 在给力时间内的RingBufferProcessor，拥有两个消费相同队列的Subscriber，允许增量消费直到RingBuffer已满。当蓝色块的下一个块是黄色块时这种情况会发生。](http://projectreactor.io/docs/reference/images/RBP.png)
**图7. 在给力时间内的`RingBufferProcessor`，拥有两个消费相同队列的Subscriber，允许增量消费直到RingBuffer已满。当蓝色块的下一个块是黄色块时这种情况会发生。**

使用帮助类的静态方法来创建`RingBufferProcessor`。

```
Processor<Integer, Integer> p = RingBufferProcessor.create("test", 32); (1)
Stream<Integer> s = Streams.wrap(p); (2)

s.consume(i -> System.out.println(Thread.currentThread() + " data=" + i)); (3)
s.consume(i -> System.out.println(Thread.currentThread() + " data=" + i)); (4)
s.consume(i -> System.out.println(Thread.currentThread() + " data=" + i)); (5)

input.subscribe(p); (6)
```

1. 创建一个Processor，它内部有一个包含32个slot的RingBuffer。
2. 基于`Reactive Stream Processor`创建`Reactor Stream`。
3. 每个请求调用consume方法在自己的线程内创建一个Disruptor的EventProcessor。
4. 每个请求调用consume方法在自己的线程内创建一个Disruptor的EventProcessor。
5. 每个请求调用consume方法在自己的线程内创建一个Disruptor的EventProcessor。
6. 由一个`Reactive Streams Publisher`订阅这个`Processor`。

传递到`Processor`中`Subscribe.onNext(Buffer)`方法的每个数据元素都会被广播给所有的消费者。这个Processor没有使用轮询分发因为这种是`RingBufferWorkProcessor`有的，我们将稍后讨论。如果你传递1，2，3三个整数到`Processor`，你会在终端看到类似的输出：

```
Thread[test-2,5,main] data=1
Thread[test-1,5,main] data=1
Thread[test-3,5,main] data=1
Thread[test-1,5,main] data=2
Thread[test-2,5,main] data=2
Thread[test-1,5,main] data=3
Thread[test-3,5,main] data=2
Thread[test-2,5,main] data=3
Thread[test-3,5,main] data=3```

每个线程都能接收到传递给Processor的所有数据，每个线程都按照数据传入的顺序接收到这些数据，因为内部使用RingBuffer来管理这些数据。
Each thread is receiving all values passed into the Processor and each thread gets the values in an ordered way since it’s using the RingBuffer internally to manage the slots available to publish values.

> RingBufferProcessor can replay missed signals -0 subscribers- to any future subscribers. That will force a processor to wait onNext() if a full buffer is not being drained by a subscriber. From the last sequence acknowledged by a subsUp to the size of the configured ringbuffer will be kept ready to be replayed for every new subscriber, even if the event has already been sent (FanOut).
