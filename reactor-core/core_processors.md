# Core Processors
Core Processor用来做比Dispatcher更集中的任务：支持背压计算异步任务。

它们可以与其他Reactive Stream提供者一起很好的工作，因为它是`org.reactivestreams.Processor`接口的直接实现。请记住Processor既是Subscriber也是Publisher，所以你可以把它插入Reactive Stream链的任何位置（source, processing, sink）。

> 该规范不建议直接调用`Processor.onNext(d)`。我们也提供了支持，但是背压除非最终被阻断当然也不会继续传播。你可以显示的使用匿名订阅传递得到背压反馈，通过使用`Processor.onSubscribe`的Processor。

> 同一时间来自单个线程的OnNext必须被序列化（不允许并发的调用onXXX）。然而如果使用协定的`Processor.share()`方法，例如：`RingBufferProcessor.share()`，Reactor也支持并发。这种决定必须耗费大量时间来使用正确的协调逻辑用于实现，所以做明智的选择：是否使用不支持并发作为标准的生产队列还是要被多线程伤得体无全肤？

***当需要特别的 XXXX Work Processor功能时，Reactor有一个标准之外的例外：***

* Reactive Streams Processor通常会在给定时间内异步的分配同样连续的数据给所有的订阅的Subscriber。类似**生产者/消费者**模型。
* **WorkProcessors**会根据它的便利性分发数据给大部分独立的Subscriber。这就是说Subscriber在指定时间内一直可以看到不同的数据。类似**工作队列**模型

我们希望在2.x release中增加我们的Core Processor集合。
