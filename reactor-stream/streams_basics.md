
## Stream基础

Reactor提供基于Reactive Streams标准的`Stream`或`Promise`来组建静态类型的数据管道。

这是一个非常有用和灵活的组件。它就像RxJava中的`Observable`一样足够灵活，可以简单的将异步行为构建在一起。但是它又足够强大，它可以像一个可随时加入和取出的异步工作队列或来自其他实现者依据标准实现的Reactive Streams组件来工作。

**本质上将有两种类型的Stream**

* 一个热的Stream是无界的，并且可以像水槽一样接收输入的数据。
 * 想象一下UI事件，例如鼠标点击或者类似Sensors、交易流或者Twitter的实时反馈。
 * 适用于Reactive Streams协议混合的背压策略。
* 一个冷的Stream是有界的，一般通过类似List或者其他迭代器保存的数据集合来创建。
 * 想象下类似IO读或者数据库查询的游标读场景。
 * 可接受自动的Reactive Streams背压策略。

> 如先前所看，Reactor使用一个JVM（和类加载器）中的Environment来保存Dispatcher实例的集合来做共享。一个 static {
  Environment.initialize();
} Environment实例可以被创建并且在应用内传递，避免类加载器隔离问题，有一个可被使用的静态类。通过这个站点的示例，我们将会使用静态帮助类并且鼓励你也这样使用。你将需要在你应用中的任意地方初始化这个静态的Environment。

> ```static {
  Environment.initialize();
}```