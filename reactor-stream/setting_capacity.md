
### Setting Capacity

The Reactive Streams standard encourages application developers to set reasonable limits on in-flight data. This prevents components from becoming inundated with more data than they can handle, which causes unpredictable problems throughout an application. One of the core concepts of Reactive Streams is that of "backpressure", or the ability of a pipeline to communicate to upstream components that it can only handle a fixed number of items at a time. A useful term to describe this process of queueing and requesting small chunks of a large volume of data is "microbatching".

Within a Reactor Stream, it’s possible to microbatch items to limit the amount of data in-flight at any given time. This has distinct advantages in a number of ways, not the least of which is that it limits exposure to data loss by preventing the system from accepting more data than it can afford to lose if the system was to crash.

To limit the amount of data in-flight in a Stream, use the .capacity(long) method.

**Streams.just()**

```
Stream<String> st;

st
  .dispatchOn(sharedDispatcher())
  .capacity(256) (1)
  .consume(s -> service.doWork(s)); (2)
```

1. Limit the amount of data in-flight to no more than 256 elements at a time.
1. Produce demand upstream by requesting the next 256 elements of data.

> capacity will not affect consume actions if the current Stream dispatcher set with dispatchOn is a SynchronousDispatcher.INSTANCE (default if unset).

> We leave as an exercise to the Reactor User to study the benefit of setting capacity vs computing dynamic demand with Stream.adaptiveConsume or a custom Subscriber.

