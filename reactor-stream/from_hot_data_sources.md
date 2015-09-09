
### From Hot Data Sources

If you are dealing with an unbounded stream of data like what would be common with a web application that accepts user input via a REST interface, you probably want to use the "hot" variety of Stream in Reactor, which we call a Broadcaster.

To use it, you simply declare a pipeline of composable, functional tasks on the Broadcaster and later call Broadcaster.onNext(T) to publish values into the pipeline.

> Broadcaster is a valid Processor and Consumer. It’s possible to onSubscribe a Broadcaster as it’s also possible to use it as a Consumer delegating Consumer.accept(T) to Broadcaster.onNext(T).

**Broadcaster.create()**

```
Broadcaster<String> sink = Broadcaster.create(Environment.get()); (1)

sink.map(String::toUpperCase) 
    .consume(s -> System.out.printf("%s greeting = %s%n",(2) Thread.currentThread(), s)); (3)

sink.onNext("Hello World!"); (4)
```

1. Create a Broadcaster using the default, shared RingBufferDispatcher as the Dispatcher.
1. Transform the input using a commonly-found convention: the map() method.
1. .consume() is a "terminal" operation, which means it produces demand in Reactive Streams parlance.
1. Publish a value into the pipeline, which will cause the tasks to be invoked.

> Hot Data Sources will never be replayed. Subscribers will only see data from the moment they have been passed to Stream.subscribe(Subscriber). An exception applies for BehaviorBroadcaster (last emitted element is replayed); Streams.timer() and Streams.period() will also maintain unique timed cursors but will still ignore backpressure.

> Subscribers will see new data N flowing through a Broadcaster every $$T+I^N$$ only after they have subscribed at time T.

**Table 7. Creating flexible Streams**

|Factory|Input|Output|
|-------|-----|------|
|Role|||
|Streams.timer(delay, unit, timer)|N/A|Long|
|Start a Timer on Stream.subscribe(Subscriber) call and emit a single onNext(0L) then onComplete() once the delay is elapsed. Be sure to pass the optional argument Timer if there is no current active Environment. Subscription.request(long)will be ignored as no backpressure can apply to a scheduled emission.|||
|Streams.period(period, unit, timer)|N/A|Long|
|Start a Timer on Stream.subscribe(Subscriber) call and every period of time emit onNext(N) where N is an incremented counter starting from 0. Be sure to pass the optional argument Timer if there is no current active Environment.Subscription.request(long) will be ignored as no backpressure can apply to a scheduled emission.|||
|Streams.<T>switchOnNext()|Publisher<T>|T|
|An Action which for the record is also a Processor. The onNext(Publisher<T>) signals will result in downstreamSubscriber<T> receiving the next Publisher sequence of onNext(T). It might interrupt a current upstream emission when theonNext(Publisher<T>) signal is received.|||
|Broadcaster.<T>create(Environment, Dispatcher)|T|T|
|Create a hot bridge between any context allowed to call onSubscribe, onNext, onComplete or onError and a composable sequence of these signals under a Stream. If no subscribers are actively registered, next signals might trigger a CancelException. The optionalDispatcher and Environment arguments define where to emit each signal. Finally, a Broadcaster can be subscribed any time to aPublisher, like a Stream.|||
|SerializedBroadcaster.create(Environment, Dispatcher)|T|T|
|Similar to Broadcaster.create() but adds support for concurrent onNext from parallel contexts possibly calling the same broadcaster onXXX methods.|||
|BehaviorBroadcaster.create(Environment, Dispatcher)|T|T|
|Simlar to Broadcaster.create() but always replays the last data signal (if any) and the last terminal signal (onComplete(),onError(Throwable)) to the new Subscribers.|||
|BehaviorBroadcaster.first(T, Environment, Dispatcher)|T|T|
|Similar to BehaviorBroadcaster but starts with a default value T.|||
|Streams.wrap(Processor<I, O>)|I|O|
|A simple delegating Stream to the passed Publisher.subscribe(Subscriber<O>) argument. Only supports well formedPublishers correctly using the Reactive Streams protocol:|||
||||
|onSubscribe > onNext\* > (onError or onComplete)|||
|Promises.<T>prepare(Environment, Dispatcher)|T|T|
||||
|Promises.ready()|||
|Prepare a Promise ready to be called exactly once by any external context through onNext. Since it’s a stateful container holding the result of the fulfilled promise, new subscribers will immediately run on the current thread.|||