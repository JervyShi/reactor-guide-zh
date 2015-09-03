# RingBufferProcessor
Reactor的`RingBufferProcessor`组件本质上是一个改造过的适应Reactive Streams API的[Disruptor RingBuffer](https://github.com/LMAX-Exchange/disruptor)。设计目的是尽可能和原生效率一样高。使用场景：你需要分发任务到其他线程，并使用极少的资源和极高的吞吐量而且可以在你的工作流中管理背压。

> 我使用`RingBufferProcessor`来计算异步的远程调用各种输出：AMQP，SSD存储和内存存储，可变的延迟几乎全部被`Processor`消灭，我们每秒百万消息的数据源从未阻塞。
> - 开心的Reactor使用者
> `RingBufferProcessor`用例

![Figure 7. RingBufferProcessor at a given time T, with 2 Subscribers, all consuming the same sequence, but delta consuming rate is allowed until the ring buffer is full. This will happen when blue cube is colliding with its next clockwise yellow cube.](http://projectreactor.io/docs/reference/images/RBP.png)
**Figure 7. RingBufferProcessor at a given time T, with 2 Subscribers, all consuming the same sequence, but delta consuming rate is allowed until the ring buffer is full. This will happen when blue cube is colliding with its next clockwise yellow cube.**

To create a RingBufferProcessor, you use static create helper methods.

```
Processor<Integer, Integer> p = RingBufferProcessor.create("test", 32); (1)
Stream<Integer> s = Streams.wrap(p); (2)

s.consume(i -> System.out.println(Thread.currentThread() + " data=" + i)); (3)
s.consume(i -> System.out.println(Thread.currentThread() + " data=" + i)); (4)
s.consume(i -> System.out.println(Thread.currentThread() + " data=" + i)); (5)

input.subscribe(p); (6)
```

1. Create a Processor with an internal RingBuffer capacity of 32 slots.
1. Create a Reactor Stream from this Reactive Streams Processor.
1. Each call to consume creates a Disruptor EventProcessor on its own Thread.
1. Each call to consume creates a Disruptor EventProcessor on its own Thread.
1. Each call to consume creates a Disruptor EventProcessor on its own Thread.
1. Subscribe this Processor to a Reactive Streams Publisher.

Each element of data passed to the Processor’s Subscribe.onNext(Buffer) method will be "broadcast" to all consumers. There’s no round-robin distribution with this Processor because that’s in the RingBufferWorkProcessor, discussed below. If you passed the integers 1, 2 and 3 into the Processor, you would see output in the console similar to this:

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

Each thread is receiving all values passed into the Processor and each thread gets the values in an ordered way since it’s using the RingBuffer internally to manage the slots available to publish values.

> RingBufferProcessor can replay missed signals -0 subscribers- to any future subscribers. That will force a processor to wait onNext() if a full buffer is not being drained by a subscriber. From the last sequence acknowledged by a subsUp to the size of the configured ringbuffer will be kept ready to be replayed for every new subscriber, even if the event has already been sent (FanOut).
