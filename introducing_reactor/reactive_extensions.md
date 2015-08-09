# Reactive扩展

Reactive扩展或者说Rx，是一组定义完备的功能性API，这组API扩展观察者模式到了史诗级的程度。

**Rx模式使用极少的设计关键字来实现对Reactive数据序列的处理：**

* 使用回调链来抽象实时及延迟：当数据可用是调用。
* 抽象不在使用的线程模型： 同步或者异步仅仅是我们处理的Observable/Stream。
* 控制错误传递和停止：错误和完成信号和数据负载信号被传递到链中。
* 在各种预先定义的API中解决了多个扩展-聚合和其他问题。

在JVM中Reactive扩展的标准实现是RxJava。它提供了一组功能丰富的API和开发实现了几乎全部的Mricrosoft原生库的概念。

Reactor 2 提供了一个特定模块实现了Reactive扩展的一部分功能。建议需要使用Reactive stream全部功能的用户使用RxJava。最后，当组合完整的RxJava系统时，用户可以从Reactor提供的强大的异步和IO的中获益。

> Some operations, behaviors, and the immediate understanding of Reactive Streams are still unique to Reactor as of now and we will try to flesh out the unique features in the appropriate section.

> Async IO capabilities are also depending on Stream Capacity for backpressure and auto-flush options.

**Table 2. Rx和Reactor Streams的对比**

rx|reactor-stream|Comment
--|--------------|-------
Observable|reactor.rx.Stream|Reflect the implementation of the Reactive Stream Publisher
Operator|reactor.rx.action.Action|Reflect the implementation of the Reactive Stream Processor
Observable with 1 data at most|reactor.rx.Promise|Type a unique result, reflect the implementation of the Reactive Stream Processor and provides for optional asynchronous dispatching.
Factory API (just, from, merge….)|reactor.rx.Streams|Aligned with a core data-focused subset, return Stream
Functional API (map, filter, take….)|reactor.rx.Stream|Aligned with a core data-focused subset, return Stream
Schedulers|reactor.core.Dispatcher, org.reactivestreams.Processor|Reactor Streams compute operations with unbounded shared Dispatchers or bounded Processors
Observable.observeOn()|Stream.dispatchOn()|Just an adapted naming for the dispatcher argument


