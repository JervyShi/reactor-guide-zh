
## Stream基础

Reactor提供基于Reactive Streams标准的`Stream`或`Promise`来组建静态类型的数据管道。

这是一个非常有用和灵活的组件。它就像RxJava中的`Observable`一样足够灵活，可以简单的将异步行为构建在一起。但是它又足够强大，它可以像一个可随时加入和取出的异步工作队列或来自其他实现者依据标准实现的Reactive Streams组件来工作。

**There are basically two rough categories of streams**

* A hot Stream is unbounded and capable of accepting input data like a sink.
 * Think UI events such as mouse clicks or realtime feeds such as sensors, trade positions or Twitter.
 * Adapted backpressure strategies mixed with the Reactive Streams protocol will apply
* A cold Stream is bounded and generally created from a fixed collection of data like a List or other Iterable.
 * Think Cursored Read such as IO reads, database queries,
 * Automatic Reactive Streams backpressure will apply

> *As seen previously, Reactor uses an Environment to keep sets of Dispatcher instances around for shared use in a given JVM (and classloader). Anstatic {
  Environment.initialize();
} Environment instance can be created and passed around in an application to avoid classloading segregation issues or the static helpers can be used. Throughout the examples on this site, we’ll use the static helpers and encourage you to do likewise. To do that, you’ll need to initialize the static Environment somewhere in your application.*

> ```static {
  Environment.initialize();
}```