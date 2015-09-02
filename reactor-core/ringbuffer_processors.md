# RingBuffer Processors

基于RingBuffer的Reactive Stream Processor优点如下：

* 高吞吐量
* 回放最近未消费的数据
 * 如果没有Subscriber监听，数据不会丢失（不像Reactor Stream中的Broadcaster会丢失数据）。
 * 如果Subscriber在处理中被中断，信号会被安全的回放，实际上它能在`RingBufferWorkProcessor`上很好的工作。
* 灵活的背压，它允许任何时间有限数量数据的背压，Subscriber有能力消费掉并且请求更多
* Smart back-pressure, it allows for a bounded size anytime and the subscribers have the responsibility to consume and request more data
* Propagated back-pressure, since it’s a Processor, it can be subscribed to pass the information along
* Multi-threaded inbound/outbound Processor capabilities

**Actually RingBuffer*Processor are like typed MicroMessageBroker !**

Their only drawbacks is they might be costly at runtime to create and they can’t be shared as easily as its cousin RingBufferDispatcher. That makes them suited for High-Throughput Pre-defined Data Pipelines.

