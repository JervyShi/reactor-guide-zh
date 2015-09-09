
## From Existing Reactive Publishers

Existing Reactive Streams **Publishers** can very well be from other implementations, including the user ones, or from Reactor itself.

The use cases incude:

* Combinatory API to coordinate various data sources.
* Lazy resource access, reading a Data Source on subscribe or on request, e.g. Remote HTTP calls.
* Data-oriented operations such as Key/Value Tuples Streams, Persistent Streams or Decoding.
* Plain Publisher decoration with Stream API

**Streams.concat() and Streams.wrap() in action**

```
Processor<String,String> processor = RingBufferProcessor.create();

Stream<String> st1 = Streams.just("Hello "); (1)
Stream<String> st2 = Streams.just("World "); (1)
Stream<String> st3 = Streams.wrap(processor); (2)

Streams.concat(st1, st2, st3) (3)
  .reduce( (prev, next) -> prev + next ) (4)
  .consume(s -> System.out.printf("%s greeting = %s%n", Thread.currentThread(), s)); (5)

processor.onNext("!");
processor.onComplete();
```

1. Create a Stream from a known value.
1. Decorate the core processor with Stream API. Note that Streams.concat() would have accepted the processor directly as a valid Publisher argument.
1. Concat the 3 upstream sources (all st1, then all st2, then all st3).
1. Accumulate the input 2 by 2 and emit the result on upstream completion, after the last complete from st3.
1. Produce demand on the pipeline, which means "start processing now".

**Table 6. Creating from available Reactive Streams Publishers**

| Factory method | Data Type |
|----------------|-----------|
| Role |  |
| Streams.create(Publisher<T>) | T |
| Only subscribe to the passed Publisher when the first Subscription.request(N) hits the returned Stream. Therefore it supports malformed Publishers that do not invoke Subscriber.onSubscribe(Subscription) as required per specification. |  |
| Streams.wrap(Publisher<T>) | T |
| A simple delegating Stream to the passed Publisher.subscribe(Subscriber<T>) argument. Only supports well formedPublishers correctly using the Reactive Streams protocol: |  |
| onSubscribe > onNext\* > (onError | onComplete) |  |
| Streams.defer(Supplier<Publisher<T>>) | T |
| A lazy Publisher access using the level of indirection provided by Supplier.get() everytime Stream.subscribe(Subscriber) is called. |  |
| Streams.createWith(BiConsumer<Long,SubscriberWithContext<T, C>,Function<Subscriber<T>,C>, Consumer<C>) | T |
| A Stream generator with explicit callbacks for each Subscriber request, start and stop events. similar toStreams.create(Publisher) minus the boilerplate for common use. |  |
| Streams.switchOnNext(Publisher<Publisher<T>>) | T |
| A Stream alterning in FIFO order between emitted onNext(Publisher<T>) from the passed Publisher. The signals will result in downstream Subscriber<T> receiving the next Publisher sequence of onNext(T). It might interrupt a current upstream emission when the onNext(Publisher<T>) signal is received. |  |
| Streams.concat(Publisher<T>, Publisher<T>*) | T |
| Streams.concat(Publisher<Publisher<T>>) |  |
| If a Publisher<T> is already emitting, wait for it to onComplete() before draining the next pending Publisher<T>. As the name suggests its useful to concat various datasources and keep ordering right. |  |
| Streams.merge(Publisher<T>, Publisher<T>, Publisher<T>*) | T |
| Streams.merge(Publisher<Publisher<T>>) |  |
| Accept multiple sources and interleave their respective sequence. Order won’t be preserved like with concat. Demand from a Subscriber will be split between various sources with a minimum of 1 to make sure everyone has a chance to send something. |  |
| Streams.combineLatest(Publisher<T1>, Publisher<T2>, Publisher<T3-N> x6,Function<Tuple2-N, C>) | C |
| Combine most recent emitted elements from the passed sources using the given aggregating Function. |  |
| Streams.combineLatest(Publisher<T1>, Publisher<T2>, Publisher<T3-N> x6,Function<Tuple2-N, C>) | C |
| Combine most recent elements once, every time a source has emitted a signal, apply the given Function and clear the temporary aggregate. Effectively it’s a flexible join mechanism for multiple types of sources. |  |
| Streams.join(Publisher<T>, Publisher<T>, Publisher<T> x6) | List<T> |
| A shortcut for zip that only aggregates each complete aggregate in a List matching the order of the passed argument sources. |  |
| Streams.await(Publisher<>, long, unit, boolean) | void |
| Block the calling thread until onComplete of the passed Publisher. Optional arguments to tune the timeout and the need to request data as well can be passed. It will throw an exception if the final state is onError. |  |
| IOStreams.<K,V>persistentMap(String, deleteOnExit) | V |
| A simple shortcut over ChronicleStream constructors, a disk-based log appender/tailer. The name argument must match an existing persistent queue under /tmp/persistent-queue\[name\]. |  |
| IOStreams.<K,V>persistentMapReader(String) | V |
| A simple shortcut over ChronicleReaderStream constructors, a disk-based log tailer. The name argument must match an existing persistent queue under /tmp/persistent-queue\[name\]. |  |
| IOStreams.decode(Codec<SRC, IN, ?>, Publisher<SRC>) | IN |
| Use Codec decoder to decode the passed source data type into IN type. |  |
| BiStreams.reduceByKey(Publisher<Tuple2<KEY,VALUE>>, Map<KEY,VALUE>, Publisher<MapStream.Signal<KEY, VALUE>>, BiFunction<VALUE, VALUE, VALUE>) | Tuple2<KEY,VALUE> |
| A key-value operation that accumulates computed results for each 2 sequential onNext(VALUE) passed to the BiFunction argument. The result will be released onComplete() only. The options allow to use an existing map store and listen for its events. |  |
| BiStreams.scanByKey(Publisher<Tuple2<KEY,VALUE>>, Map<KEY,VALUE>, Publisher<MapStream.Signal<KEY, VALUE>>, BiFunction<VALUE, VALUE, VALUE>) | Tuple2<KEY,VALUE> |
| A key-value operation that accumulates computed results for each 2 sequential onNext(VALUE) passed to the BiFunction argument. The result will be released every time just after it has been stored. The options allow you to use an existing map store and listen for its events. |  |
| Promises.when(Promise<T1>, Promise<T2>, Promise<T3-N> x6) | TupleN<T1,T2,\*?> |
| Join all unique results from Promises and provide for the new Promise with the aggregated Tuple. |  |
| Promises.any(Promise<T>, Promise<T>, Promise<T> x6) | T |
| Pick the first signal available among the passed promises and onNext(T) the returned Promise with this result. |  |
| Promises.multiWhen(Promise<T>, Promise<T>, Promise<T> x6) | List<T> |
| Join all unique results from Promises and provide for the new Promise with the aggregated List. The difference with the whenalternative is that the type of promises must match. |  |