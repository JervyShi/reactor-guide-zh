# Reactive Streams

Reactive Streams是一个新的标准, 被不同的厂商和技术组织支持，包括Netflix、Oracle、 Pivotal 和Typesafe。 期望被Java 9或以后的版本收录。

该标准的目标是提供一种同步或者异步的具有控制流机制的数据序列。此标准十分轻量并且把第一目标放在JVM上。它提供4个Java接口，一个TCK和一些例子。这4个接口的实现非常直接，但这个项目的内涵是由TCK校验的行为。生产者是有资格实现Reactive Streams的，很幸运我们已经做了一些可以通过TCK校验的实现类来让生产者有资格实现Reactive Streams标准。

![图 3. Reactive Streams合约](http://projectreactor.io/docs/reference/images/rs.png)
**图 3. Reactive Streams合约**

**Reactive Streams接口**

* org.reactivestreams.Pubslisher: 数据源 （从0到N，其中N可以是无限的）。 它提供了两个可选的结束事件：error和completion。
* org.reactivestreams.Subscriber: 消费者数据序列（从0到N，其中N可以是无限的）。它这启动时获取一个subscription来确定接下来需要处理多少数据。与其它数据时序信号交互的其它回调有: next (新消息)和可选的completion/error。
* org.reactivestreams.Subscription: 在初始化时传递给Subscriber的一个小追踪器。它控制有多少数据是我们准备去消费多少数据和什么时候停止消费（取消）。
* org.reactivestreams.Processor: 既是Subscriber又是Publisher的一个组件标记。

![图 4. Reactive Streams发布协议
](http://projectreactor.io/docs/reference/images/signals.png)
**图 4. Reactive Streams发布协议**

**通过传递Subscription，一个Subscriber从一个Publisher请求数据有两种方式：**

* 无限制的: 在订阅时, 仅调用`Subscription`的`request(Long.MAX_VALUE)`方法。
* 有限制的: 在订阅时, 保留`Subscription`的引用，并且当`Subscriber`准备处理数据时调用`request(long)`方法。
 * 通常情况下，在订阅时Subscribers将请求一组初始数据或者甚至1个数据。
 * 然后在`onNext`已被认定调用成功后（例如：Commit后, Flush 后等等…)，请求更多数据。
 * 建议使用线性数量的请求。尽量避免重复请求，例如： 每个下次请求时请求10个或者更多数据。


表 1. 目前为止，Reactor直接使用的Reactive Streams接口及实现：

Reactive Streams|Reactor Module(s)|Implementation(s)|Description
----------------|-----------------|-----------------|-----------
Processor|reactor-core, reactor-stream|reactor.core.processor.*, reactor.rx.*|In Core, we offer backpressure-ready RingBuffer*Processor and more, in Stream we have a full set of Operations and Broadcasters.
Publisher|reactor-core, reactor-bus, reactor-stream, reactor-net|reactor.core.processor.*, reactor.rx.stream.*, reactor.rx.action.*, reactor.io.net.*|In Core, processors implement Publisher. In Bus we publish an unbounded emission of routed events. In Stream, our Stream extensions directly implement Publisher. In Net, Channels implement Publisher to consume incoming data, we also provide publishers for flush and close callbacks.
Subscriber|reactor-core, reactor-bus, reactor-stream, reactor-net|reactor.core.processor.*, reactor.bus.EventBus.*, reactor.rx.action.*, reactor.io.net.impl.*|In Core, our processor implement Subscriber. In Bus, we expose bus capacities with unbounded Publisher/Subscriber. In Stream, actions are Subscribers computing specific callbacks. In Net, our IO layer implements subscribers to handle writes, closes and flushes.
Subscription|reactor-stream, reactor-net|reactor.rx.subscription.*, reactor.io.net.impl.*|In Stream, we offer optimized PushSubscriptions and buffering-ready ReactiveSubscription. In Net, our Async IO reader-side use custom Subscriptions to implement backpressure.

从reactor 2启动时我们就一直遵循这个标准，并且随着标准的改变而改变直到1.0.0正式版准备发布。现在可以通过maven中央库及其它流行的镜像上可以找到该标准，你将发现它依赖reactor-core模块作为过度。

