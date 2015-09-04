# RingBufferWorkProcessor

`RingBufferWorkProcessor`不像标准的`RingBufferProcessor`会广播数据给所有消费者，`RingBufferWorkProcessor`根据进入的数据基于消费者的数量做分区。流入`Processor`的数据会基于轮询机制发送到多个线程中（因为每个消费者拥有自己的线程），依旧使用`RingBuffer`来管理数据的发布。

> 我们实现了一个`RingBufferWorkProcessor`，可扩展、支持负载均衡的各种HTTP微服务调用。我可能是错的，但是它看起来快过光速并且GC压力完全在可控范围内。

> — Happy Reactor user

> Use Case for RingBufferWorkProcessor

![Figure 8. RingBufferWorkProcessor at a given time T, with 2 Subscribers, each consuming unique sequence (availabilty FIFO), delta consuming rate is allowed until the ring buffer is full. This will happen when blue cube is colliding with its next clockwise yellow cube.](http://projectreactor.io/docs/reference/images/RBWP.png)
**Figure 8. RingBufferWorkProcessor at a given time T, with 2 Subscribers, each consuming unique sequence (availabilty FIFO), delta consuming rate is allowed until the ring buffer is full. This will happen when blue cube is colliding with its next clockwise yellow cube.**

To use the `RingBufferWorkProcessor`, the only thing you have to change from the above code sample is the reference to the static `create` method. You’ll use the one on the `RingBufferWorkProcessor` class itself instead. The rest of the code remains identical.

```
Processor<Integer, Integer> p = RingBufferWorkProcessor.create("test", 32);(1)
```
1. Create a Processor with an internal RingBuffer capacity of 32 slots.

Now when values are published to the `Processor`, they will not be broadcast to every consumer, but be partitioned based on the number of consumers. When we run this sample, we see output like this now:

```
Thread[test-2,5,main] data=3
Thread[test-3,5,main] data=2
Thread[test-1,5,main] data=1
```

> RingBufferWorkProcessor can replay interrupted signals, detecting CancelException from the terminating subscriber. It will be the only case where a signal will actually be played eventually once more with another Subscriber. We guarantee at-least-once delivery for any events. If you are familiar with semantic you might now say "Wait, this RingBufferWorkProcessor works like a Message Broker?", and the answer is yes.
