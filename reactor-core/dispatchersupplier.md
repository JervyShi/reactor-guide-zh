# DispatcherSupplier
You may have noticed some Dispatchers are single-threaded, especially the RingBufferDispatcher and MpscDispatcher. Going further, refering to the Reactive Stream specification, the Subscriber/Processor implementation should not allow for concurrent notifications. This impacts Reactor Streams in particular, and trying to use Stream.dispatchOn(Dispatcher) with a Dispatcher that leaves the door open to concurrent signals will fail explicitely.

There is however a way to workaround that limitation by using pools of Dispatcher or DispatcherSupplier. In effect, as a Supplier factory, the indirection offered by Supplier.get() to retrieve a Dispatcher allow for interesting pooling strategy : RoundRobin, Least-Used…​

Environment offers static helpers to create, and eventually register against the current active Environment pools of Dispatchers: groups of RoundRobin returned Dispatchers. Once ready, suppliers will provide for a controlled number of Dispatchers.

As usual with Dispatchers, Environment is the one-stop shop to manage them:

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

