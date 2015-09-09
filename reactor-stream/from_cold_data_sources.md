
### From Cold Data Sources

You can create a `Stream` from a variety of sources, including an `Iterable` of known values, a single value to use as the basis for a flow of tasks, or even from blocking structures such as `Future` of `Supplier`.

**Streams.just()**

```
Stream<String> st = Streams.just("Hello ", "World", "!"); (1)

st.dispatchOn(Environment.cachedDispatcher()) (2)
  .map(String::toUpperCase) (3)
  .consume(s -> System.out.printf("%s greeting = %s%n", Thread.currentThread(), s)); (4)
```

1. Create a Stream from a known value but do not assign a default Dispatcher.
1. .dispatchOn(Dispatcher) tells the Stream which thread to execute tasks on. Use this to move execution from one thread to another.
1. Transform the input using a commonly-found convention: the map() method.
1. Produce demand on the pipeline, which means "start processing now". It’s an optimize shortcut for subscribe(Subscriber) where the Subscriber just requests Long.MAX_VALUE by default.

> *Cold Data Sources will be replayed from start for every fresh Subscriber passed to `Stream.subscribe(Subscriber)`, and therefore duplicate consuming is possible.*

**Table 5. Creating pre-determined Streams and Promises**

|Factory method|Data Type|
|--------------|---------|
|**Role**||
|Streams.<T>empty()|T|
|Only emit onComplete() once requested by its Subscriber.||
|Streams.<T>never()|T|
|Never emit anything. Useful for keep-alive behaviors.||
|Streams.<T, Throwable>fail(Throwable)|T|
|Only emit onError(Throwable).||
|Streams.from(Future<T>)|T|
|Block the Subscription.request(long) on the passed Future.get() that might emit onNext(T) and onComplete()otherwise onError(Throwable) for any exception.||
|Streams.from(T[])|T|
|Emit N onNext(T) elements everytime Subscription.request(N) is invoked. If N == Long.MAX_VALUE, emit everything. Once all the array has been read, emit onComplete().||
|Streams.from(Iterable<T>)|T|
|Emit N onNext(T) elements everytime Subscription.request(N) is invoked. If N == Long.MAX_VALUE, emit everything. Once all the array has been read, emit onComplete().||
|Streams.range(long, long)|Long|
|Emit a sequence of N onNext(Long) everytime Subscription.request(N) is invoked. If N == Long.MAX_VALUE, emit everything. Once the inclusive upper bound been read, emit onComplete().||
|Streams.just(T, T, T, T, T, T, T, T)|T|
|An optimization over Streams.from(Iterable) that just behaves similarly. Also useful to emit Iterable, Array or Future without colliding with the Streams.from() signatures.||
|Streams.generate(Supplier<T>)|T|
|Emit onNext(T) from the producing Supplier.get() factory everytime Subscription.request(N) is called. The demand N is ignored as only one data is emitted. When a null value is returned, emit onComplete().||
|Promises.syncTask(Supplier<T>), Promises.task(, Supplier<T>)|T|
|Emit a single onNext(T) and onComplete() from the producing Supplier.get() on the first Subscription.request(N)received. The demand N is ignored.||
|Promises.success(T)|T|
|Emit onNext(T) and onComplete() whenever a Subscriber is provided to Promise.subscribe(Subscriber).||
|Promises.<T>error(Throwable)|T|
|Emit onError(Throwable) whenever a Subscriber is subscribed is provided to Promise.subscribe(Subscriber).||