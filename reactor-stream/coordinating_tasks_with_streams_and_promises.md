
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
* Action和Stream提供的所有操作的直接实现都遵循Reactive Stream规范
 * 不直接创建，但是使用Stream API（Stream.map() → MapAction, Stream.filter() → FilterAction…） 
* Broadcaster，一种特殊的Action为动态数据分发暴露onXXXX接口
 * 不同于Core Processor，当没有消费者注册时他们不会犹豫是否缓存数据
 * 然而BehaviorBroadcaster可以重播最近的信号给新的Subscriber们

> *不要混淆```reactor.rx.Stream```和新的JDK 8``` java.util.stream.Stream```。后者不提供基于Reactive Streams 的API或者Reactive扩展。然而，JDK8的Stream API是相当完整的适用于原始类型和集合的使用。实际上，JDK 8允许应用混合JDK和Reactive Stream还是挺有趣的。*
