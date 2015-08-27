# DispatcherSupplier
你可能注意到了一些Dispatcher是单线程的，特别是RingBufferDispatcher和MpscDispatcher。深入来说，根据Reactive Stream的说明，Subscriber和Processor的实现是不允许并发通知的。这一点对Reactor Stream有极其特殊的影响，试图用Stream.dispatchOn(Dispatcher)与Dispatcher来关闭可使用并发的大门。

然而还有一种方式可以解决这种限制，使用Dispatcher pool或者DispatcherSupplier。实际上作为一个Supplier工厂，Supplier.get()方法根据有趣的pooling(XX：池/共享)策略：轮询、最少使用。。等间接提供一个Dispatcher。

Environment提供一个静态帮助类来创建并注册到当前活跃的Environment和Dispatcher pool：一组轮询返回的Dispatcher集合。一旦就绪，supplier就会提供一组可控数量的Dispatcher集合。

不用与Dispatcher集合，Environment提供一站式的管理服务：

```
Environment.initialize();
//....

//Create an anonymous pool of 2 dispatchers with automatic default settings (same type than default dispatcher, default backlog size...)
DispatcherSupplier supplier = Environment.newCachedDispatchers(2);

Dispatcher d1 = supplier.get();
Dispatcher d2 = supplier.get();
Dispatcher d3 = supplier.get();
Dispatcher d4 = supplier.get();

Assert.isTrue( d1 == d3  && d2 == d4);
supplier.shutdown();

//Create and register a new pool of 3 dispatchers
DispatcherSupplier supplier1 = Environment.newCachedDispatchers(3, "myPool");
DispatcherSupplier supplier2 = Environment.cachedDispatchers("myPool");

Assert.isTrue( supplier1 == supplier2 );
supplier1.shutdown();
```

