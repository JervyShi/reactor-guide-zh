# RingBuffer Processors

基于RingBuffer的Reactive Stream Processor优点如下：

* 高吞吐量
* 回放最近未消费的数据
 * 如果没有`Subscriber`监听，数据不会丢失（不像Reactor Stream中的`Broadcaster`会丢失数据）。
 * 如果`Subscriber`在处理中被中断，信号会被安全的回放，实际上它能在`RingBufferWorkProcessor`上很好的工作。
* 灵活的背压，它允许任何时间有限数量数据的背压，`Subscriber`有能力消费掉并且请求更多数据
* 传播的背压，因为它是一个`Processor`，它可以通过订阅方式传递消息。
* Processer拥有多线程处理出入的能力

**其实RingBuffer*Processor更类似一个典型的MicroMessageBroker !**

它们唯一的缺点是运行时创建开销较大，它们不能像它们的表兄`RingBufferDispatcher`一样易于分享。这让它们更适合可预定义的高吞吐量管道。

