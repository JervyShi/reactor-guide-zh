# RingBufferWorkProcessor

`RingBufferWorkProcessor`不像标准的`RingBufferProcessor`会广播数据给所有消费者，`RingBufferWorkProcessor`根据进入的数据基于消费者的数量做分区。流入`Processor`的数据会基于轮询机制发送到多个线程中（因为每个消费者拥有自己的线程），依旧使用`RingBuffer`来管理数据的发布。

> 我们实现了一个`RingBufferWorkProcessor`，可扩展、支持负载均衡的各种HTTP微服务调用。我可能是错的，但是它看起来快过光速并且GC压力完全在可控范围内。

> — 愉快的Reactor使用者

> RingBufferWorkProcessor的使用案例

![图8. 在给力时间内的RingBufferProcessor，各自消费不同队列（消费方式：先进先出）的Subscriber，允许增量消费直到RingBuffer已满。当蓝色块的下一个块是黄色块时这种情况会发生。](http://projectreactor.io/docs/reference/images/RBWP.png)
**图8. 在给力时间内的RingBufferProcessor，各自消费不同队列（消费方式：先进先出）的Subscriber，允许增量消费直到RingBuffer已满。当蓝色块的下一个块是黄色块时这种情况会发生。**

要使用`RingBufferWorkProcessor`，唯一需要修改的就是上面代码例子中`create`的静态方法。你将会使用`RingBufferWorkProcessor`类自己的方法替代原有方法。余下代码保持一致。

```
Processor<Integer, Integer> p = RingBufferWorkProcessor.create("test", 32);(1)
```

1. 创建一个Processor，它内部有一个包含32个slot的RingBuffer。

现在当有数据被发布到`Processor`，它们将不会再广播给所有消费者，但是会被基于消费者数量进行分区。当我们运行这个例子时，我们将会类似下面的结果：

```
Thread[test-2,5,main] data=3
Thread[test-3,5,main] data=2
Thread[test-1,5,main] data=1
```

> `RingBufferWorkProcessor`能回放被中断的信号，检测正在停止工作的Subscriber的取消异常。这会是唯一一种特例，一个信号可能会被播放不至一次。我们保证任何事件最好被执行一次。如果你熟悉这个语义，你可能会说“等等，`RingBufferWorkProcessor`像一个消息Broker一样运作？”答案是肯定的。like a Message Broker?", and the answer is yes.
