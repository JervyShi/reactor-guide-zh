
## Combinatory Operations

Combining Publishers allows for coordination between multiple concurrent sequences of data. They also serve the purpose of asynchronous transformations, with the resulting sequences being merged.

Coordinating in a non-blocking way will free the developer from using Future.get() or Promise.await(), a perilous task when it comes to more than one signal. Being non-blocking means that distinct pipelines won’t wait on anything other than Subscriber demand. The Subscriber requests will be split, with a minimum request of one for each merged Publisher.

Merging actions are modeled in FanInAction and take care of concurrent signaling with a thread-stealing SerializedSubscriber proxy to the delegate Subscriber. For each signal it will verify if the correct thread is already running the delegate Subscriber and rescheduling the signal if not. The signal will then be polled when the busy thread exits Subscriber code, possibly running the signal in a different thread than originally produced on.

> Reducing the demand volume before using flatMap might be a good or a bad idea. In effect it doesn’t deserve the merging action to subscribe to many parallel Publisher if it can’t actually process them all. However it limiting the parallel Publisher size might also not give a chance to faster Publisher pending a request to be delivered.

**Stream.zipWith(Function)**

```
Streams
  .range(1, 100)
  .zipWith( Streams.generate(System::currentTimeMillis), tuple -> tuple )    (1)
  .consume(
    tuple -> System.out.println("number: "+tuple.getT1()+" time: "+tuple.getT2()) ,    (2)
    Throwable::printStackTrace,
    avoid -> System.out.println("--complete--")
  );
```

1. "Zip" or aggregate the most recent signal from RangeStream and the passed SupplierStream providing current time
1. "Zip" produces tuples of data from each zipped Publisher in the declarative order (left to right, stream1.zipWith(stream2)).

**Table 13. Combining Data Sources**

|	Functional API or Factory method	|
|----|
|	Role	|
|		|
|	Stream.flatMap(Function<T, Publisher<V>>)	|
|	An Async transformation is a typed shortcut for map(Function<T, Publisher<V>>).merge().	|
|		|
|	The mapping part produces a Publisher<V> eventully using the passed data T, a common pattern used in MicroService architecture.	|
|		|
|	The merging part transforms the sequence of produced Publisher<V> into a sequence of V by safely subscribing in parallel to all of them. There is no ordering guaranteed, it is interleaved sequence of V. All merged Publisher<T> must complete before theSubscriber<T> can complete.	|
|	Streams.switchOnNext(Publisher<Publisher<T>>)	|
|	A Stream alterning in FIFO order between emitted onNext(Publisher<T>) from the passed Publisher. The signals will result in downstream Subscriber<T> receiving the next Publisher sequence of onNext(T). It might interrupt a current upstream emission when the onNext(Publisher<T>) signal is received. All merged Publisher<T> must complete before the Subscriber<T> can complete.	|
|	Streams.merge(Publisher<T>, Publisher<T> x7)	|
|		|
|	Streams.merge(Publisher<Publisher<T>>)	|
|		|
|	Stream.mergeWith(Publisher<T>)	|
|		|
|	Stream.merge()	|
|	Transform upstream sequence of Publisher<T> into a sequence of T by safely subscribing in parallel to all of them. There is no ordering guaranteed, it is interleaved sequence of T. If the arguments are directly Publisher<T> like inStream.mergeWith(Publisher<T>) or Streams.merge(Publisher<T>, Publisher<T>), the MergeAction will subscribe to them directly and size more efficiently (known number of parallel upstreams). All merged Publisher<T> must complete before theSubscriber<T> can complete.	|
|	Streams.concat(Publisher<T>, Publisher<T> x7)	|
|		|
|	Streams.concat(Publisher<Publisher<T>>)	|
|		|
|	Stream.concatWith(Publisher)	|
|		|
|	Stream.startWith(Publisher)	|
|	Similar to merge() actions but if a Publisher<T> is already emitting, wait for it to onComplete() before draining the next pending Publisher<T>. The sequences will be subscribed in declarative order, from left to right, e.g. stream1.concatWith(stream2) or with the argument given in stream2.startWith(stream1).	|
|	Streams.combineLatest(Publisher<T>, Publisher<T> x7, Function<Tuple,V>)	|
|		|
|	Streams.combineLatest(Publisher<Publisher<T>>, Function<Tuple,V>)	|
|	Combine the most recent onNext(T) signal from each distinct Publisher<T>. Each signal combines until a future onNext(T) from its source Publisher<T> replaces it. After all Publisher<T> have emitted at least one signal, the given combinator function will accept all recent signals and produce the desired combined object. If any Publisher<T> completes, the downstream Subscriber<T>will complete.	|
|	Streams.zip(Publisher<T>, Publisher<T> x7, Function<Tuple,V>)	|
|		|
|	Streams.zip(Publisher<Publisher<T>>, Function<Tuple,V>)	|
|		|
|	Stream.zipWith(Publisher<T>, Function<Tuple2,V>)	|
|	Combine the most recent onNext(T) signal from each distinct Publisher<T>. Each signal combines only once. Everytime allPublisher<T> have emitted one signal, the given zipper function will receive them and produce the desired zipped object. If anyPublisher<T> completes, the downstream Subscriber<T> will complete.	|
|	Streams.join(Publisher<T>, Publisher<T> x7)	|
|		|
|	Streams.join(Publisher<Publisher<T>>)	|
|		|
|	Stream.joinWith(Publisher<T>)	|
|	A shortcut for zip with a predefined zipper function transforming each most recent Tuple into a List<?>.	|