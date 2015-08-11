# RingBuffer Processors
RingBuffer-based Reactive Streams Processors are good for a bunch of #awesome reasons:

* Über high throughput
* Replay from the latest not consumed data
 * If no Subscriber is listening, data won’t be lost (unlike Broadcaster from Reactor-Stream).
 * If a Subscriber cancels during processing, signal will be safely replayed, it actually works nicely with the RingBufferWorkProcessor
* Smart back-pressure, it allows for a bounded size anytime and the subscribers have the responsibility to consume and request more data
* Propagated back-pressure, since it’s a Processor, it can be subscribed to pass the information along
* Multi-threaded inbound/outbound Processor capabilities

**Actually RingBuffer*Processor are like typed MicroMessageBroker !**

Their only drawbacks is they might be costly at runtime to create and they can’t be shared as easily as its cousin RingBufferDispatcher. That makes them suited for High-Throughput Pre-defined Data Pipelines.

