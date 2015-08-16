# 核心模块概述
![图5. Doge怎样使用Reactor-Core模块](http://projectreactor.io/docs/reference/images/core-overview.png)
**Figure 5. Doge怎样使用Reactor-Core模块**

**Reactor Core** 有如下子单元:

* Common IO和函数式类型，一些是直接从Java 8函数式接口回迁的。

 * Function, Supplier, Consumer, Predicate, BiConsumer, BiFunction

 * Tuples

 * Resource, Pausable, Timer

 * Buffer, Codec and a handful of predifined Codecs

* Environment上下文

* Dispatcher协议和一组预定义的Dispatcher

* 预定义的Reactive Streams Processor

reactor-core模块可以直接用于替代已有的消息传递策略，调度定时任务或者用小巧的基于Java 8的函数式代码块来管理你的代码。这种突破使开发者更容易使用其他Reactive基础库而不需要理解RingBuffer的实现。

*Reactor-Core模块隐含了LMAX Disruptor，所以不会与现有的Disruptor产生冲突。*




