
## Backpressure and Overflow

Backpressure is addressed automatically in many situations with the Reactive Streams contract. If a Subscriber doesn’t request more than it can actually process (e.g. something other than Long.MAX_VALUE), the upstream source can avoid sending too much data. With a "cold" Publisher this only works when you can stop reading from source at any time: How much to read from a socket, How many rows from a SQL query cursor, how many lines from a File, how many elements from an Iterable…

If the source is hot, such as a timer or UI events, or the Subscriber might request Long.MAX_VALUE on a large dataset, a strategy must be explicitely picked by the developer to deal with backpressure.

**Reactor provides a set of APIs to deal with Hot and Cold sequences:**

* Uncontrolled sequences (Hot) should be actively managed
 * By reducing the sequence volume, e.g. "sampling"
 * By ignoring data when the demand exceeds capacity
 * By buffering data when the demand exceeds capacity
* Controlled sequences (Cold) should be passively managed
 * By lowering demand from the Subscriber or at any point of the Stream
 * By gapping demand with delayed requests

A common example used extensively in the Reactive Extensions documentation is the Marble Diagram. The dual timeline helps visualize when and what is observed in the Publisher or Stream and in a Subscriber (e.g. an Action). We will use these diagrams here to emphasize the demand flow, where usually such a diagram details the nature of the transformation like map or filter.

![marble-101](http://projectreactor.io/docs/reference/images/marble/marble-101.png)

Reactor will automatically provide for an in-memory overflow buffer when the dispatcher or the capacity differs from one action to another. This will not apply to Core Processors, which handle the overflow in their own way. Dispatchers can be re-used and Reactor must limit the number of dispatches where it can, hence the in-memory buffer added by Action when dispatchers differ.

```
Streams.just(1,2,3,4,5)
  .buffer(3)    (1)
  //onOverflowBuffer()
  .capacity(2)  (2)
  .consume()


Streams.just(1,2,3,4,5)
  .dispatchOn(dispatcher1)  (3)
  //onOverflowBuffer()
  .dispatchOn(dispatcher2)  (4)
  .consume()
```

1. The buffer operation set capacity(3)
1. consume() or any downstream action is set with capacity(2), an implicit onOverflowBuffer() is added
1. A first action running on dispatcher1
1. A second action running on a different dispatcher2, an implicit onOverflowBuffer() is added

Ultimately the Subscriber can request data one by one, limiting the in-flight data to one element all along the pipeline and requesting one more after each successful onNext(T). The same behavior can be obtained with capacity(1).consume(...).

```
Streams.range(1,1000000)
  .subscribe(new DefaultSubscriber<Long>(){     (1)
    Subscription sub;

    @Override
    void onSubscribe(Subscription sub){
      this.sub = sub;
      sub.request(1);   (2)
    }

    @Override
    void onNext(Long n){
      httpClient.get("localhost/"+n).onSuccess(rep -> sub.request(1));    (3)
    }
  );
```

1. Use a DefaultSubscriber to avoid implementing all Subscriber methods.
1. Schedule a first demand request after keeping a reference to the subscription.
1. Use Async HTTP API to request more only on successful GET. That will naturally propagate the latency information back to the RangeStream Publisher. One can imagine then measuring the time difference between two requests and how that gives an interesting insight into the processing and IO latency.

**Table 12. Controlling the volume of in-flight data**

|	Stream<T>	|
|-----------|
|	Role	|
|	subscribe(Subscriber<T>)	|
|	A custom Subscriber<T> will have the flexibility to request whenever it wishes. It’s best to change the size these requests if theSubscriber uses blocking operations.	|
|	capacity(long)	|
|	Set the capacity to this Stream<T> and all downstream actions.	|
|	onOverflowBuffer(CompletableQueue)	|
|	Create or use the given CompletableQueue to store the overflow elements. Overflow occurs when a Publisher sends more data than a Subscriber has actually requested. Overflow will be drained over the next calls to request(long).	|
|	![marble-overflowbuffer](http://projectreactor.io/docs/reference/images/marble/marble-overflowbuffer.png)	|
|		|
|	onOverflowDrop()	|
|	Ignore the overflowed elements. Overflow occurs when a Publisher sends more data than a Subscriber has actually requested. Overflow will be drained over the next calls to request(long).	|
|	![marble-overflowdrop](http://projectreactor.io/docs/reference/images/marble/marble-overflowdrop.png)	|
|		|
|	throttle(long)	|
|	Delay downstream request(long) and periodically decrement the accumulated demand one by one to request upstream.	|
|	![marble-throttle.png](http://projectreactor.io/docs/reference/images/marble/marble-throttle.png)	|
|		|
|	requestWhen(Function<Stream<Long>, Publisher<Long>>)	|
|	Pass any downstream request(long) to Stream<Long> sequence of requests that can be altered and returned using any form ofPublisher<Long>. The RequestWhenAction will subscribe to the produced sequence and immiately forward onNext(Long) to the upstream request(long). It behaves similarly to adaptiveConsume but can be inserted at any point in the Stream pipeline.	|
|	![marble-requestwhen](http://projectreactor.io/docs/reference/images/marble/marble-requestwhen.png)	|
|		|
|	batchConsume(Consumer<T>, Consumer<T>, Consumer<T>, Function<Long,Long>)	|
|		|
|	batchConsumeOn	|
|	Similar to consume but will request the mapped Long demand given the previous demand and starting with the defaultStream.capacity(). Useful for adapting the demand from various factors.	|
|	adaptiveConsume(Consumer<T>, Consumer<T>, Consumer<T>, Function<Stream<Long>,Publisher<Long>>),	|
|		|
|	adaptiveConsumeOn	|
|	Similar to batchConsume but will request the computed sequence of demand Long. It can be used to insert flow-control such asStreams.timer() to delay demand. The AdaptiveConsumerAction will subscribe to the produced sequence and immediately forwards onNext(Long) to the upstream request(long).	|
|	process(Processor<T, ?>)	|
|	Any Processor can also take care of transforming the demand or buffer. It is worth checking into the behavior of the specificProcessor implementation in use.	|
|	All filter(arguments), take(arguments), takeWhile(arguments)…	|
|	All limit operations can be used to proactively limit the volume of a Stream.	|
|	buffer(arguments), reduce(arguments), count(arguments)…	|
|	All aggregating and metrics operations can be used to proactively limit the volume of a Stream.	|
|	All sample(arguments), sampleFirst(arguments)	|
|	Reduce the volume of a Stream<T> by selecting the last (or the first) onNext(T) signals matching the given conditions. These conditions can be timed, sized, timed or sized, and interactive (event-driven).	|
|	zip(arguments), zipWith(arguments)	|
|	Reduce the volume of N Stream<T> to the least signals producing zipped Publisher. The aggregated signals from each Publishercan be used to produce a distinct value from the N most recent upstream onNext(T).	|