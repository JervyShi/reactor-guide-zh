
## 通过Stream和Promise协调任务

![Figure 9. Doge如何使用 Reactor-Stream](http://projectreactor.io/docs/reference/images/streams-overview.png)
**Figure 9. Doge如何使用 Reactor-Stream**

**Reactor Streams**有以下特性:

* Stream和它的直接实现
 * 包含Reactive扩展和其他组合类的API
* Promise和一组独特的A+风味的API
 * 可以使用Promise.stream()转换为Stream 
* 静态工厂，一步创建关联组件
 * Streams是用于通过明确定义数据源（Iterable, nothing, Future, Publisher…）的Stream
 * BiStreams是用于创建处理key-value的Stream<Tuple2>（reduceByKey…
 * IOStreams是用于持久化和解码的Stream
 * Promises是用于单一数据的Promise
* Action and its direct implementations of every operation provided by the Stream following the Reactive Streams Processor specification
 * Not created directly, but with the Stream API (Stream.map() → MapAction, Stream.filter() → FilterAction…)
* Broadcaster, a specific kind of Action exposing onXXXX interfaces for dynamic data dispatch
 * Unlike Core Processors, they will usually not bother with buffering data if there is no subscriber attached
 * However the BehaviorBroadcaster can replay the latest signal to new Subscribers

> *Do not confuse reactor.rx.Stream with the new JDK 8 java.util.stream.Stream. The latter does not offer a Reactive Streams based API nor Reactive Extensions. However, the JDK 8 Stream API is quite complete when used with primitive types and Collections. In fact it’s quite interesting for JDK 8 enabled applications to mix both JDK and Reactive Streams.*
