# RingBufferWorkProcessor

`RingBufferWorkProcessor`不像标准的`RingBufferProcessor`会广播数据给所有消费者，`RingBufferWorkProcessor`根据进入的数据基于消费者的数量做分区。流入`Processor`的数据会基于轮询机制发送到多个线程中（因为每个消费者拥有自己的线程），依旧使用`RingBuffer`来管理数据的发布。

> 我们实现了一个`RingBufferWorkProcessor`，可扩展、支持负载均衡的各种HTTP微服务调用。我可能是错的，但是它看起来快过光速并且GC压力完全在可控范围内。

> — 愉快的Reactor使用者

> RingBufferWorkProcessor的使用案例

![图8. 在给力时间内的RingBufferProcessor，各自消费不同队列（消费方式：先进先出）的Subscriber，允许增量消费直到RingBuffer已满。当蓝色块的下一个块是黄色块时这种情况会发生。](http://projectreactor.io/docs/reference/images/RBWP.png)
**图8. 在给力时间内的RingBufferProcessor，各自消费不同队列（消费方式：先进先出）的Subscriber，允许增量消费直到RingBuffer已满。当蓝色块的下一个块是黄色块时这种情况会发生。**

要使用`RingBufferWorkProcessor`，唯一需要修改的就是上面代码例子中`create`的静态方法。你将会使用`RingBufferWorkProcessor`类自己的方法替代原有方法。余下代码保持一致。
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
