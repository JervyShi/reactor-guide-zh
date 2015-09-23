
## 创建Streams和Promises

如果你是某个数据源的所有者，并且想要把它做成可直接访问并且包含多种Reactvie插件和Reactive Stream能力的Reactive，那么这是一个开始。

有时，它也是一种使用`Stream`API扩展现有`Reactive Stream Publisher`的示例，很幸运的是我们提供一次性的静态API来处理它们之间的转换。

使用Reactor API来通过注入方式创建`Publisher`是一个刺激的选项，就像我们通过继承现有的`Reactor Stream`来实现的`IterableStream`, `SingleValueStream`一样。

> *Streams和Promises都比较便宜，我们的微基准套件成功通过商用硬件创造了超过150M/秒的服务能力。大部分流的坚持无共享模式，仅在需要时创建新的不可变对象。*

> *每次操作都会返回一个新的实例：*

```
Stream<A> stream = Streams.just(a);
Stream<B> transformedStream = stream.map(transformationToB);

Assert.isTrue(transformationStream != stream);
stream.subscribe(subscriber1); //subscriber1 will see the data A unaltered
transformedStream.subscribe(subscriber2); //subscriber2 will see the data B after transformation from A.

//Note theat these two subscribers will materialize independant stream pipelines, a process we also call lifting
```
