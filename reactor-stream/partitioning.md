
## Partitioning

Partition a Stream for concurrent, parallel work.

An important aspect of the functional composition approach to reactive programming is that work can be broken up into discreet chunks and scheduled to run on arbitrary Dispatchers. This means you can easily compose a flow of work that starts with an input value, executes work on another thread, and then passes through subsequent transformation steps once the result is available. This is one of the more common usage patterns with Reactor.

```
DispatcherSupplier supplier1 = Environment.newCachedDispatchers(2, "groupByPool");
DispatcherSupplier supplier2 = Environment.newCachedDispatchers(5, "partitionPool");

Streams
    .range(1, 10)
    .groupBy(n -> n % 2 == 0)   (1)
    .flatMap(stream -> stream
            .dispatchOn(supplier1.get())    (2)
            .log("groupBy")
    )
    .partition(5)   (3)
    .flatMap(stream -> stream   
            .dispatchOn(supplier2.get())    (4)
            .log("partition")
    )
    .dispatchOn(Environment.sharedDispatcher())     (5)
    .log("join")
    .consume();
```

1. Create at most two streams (odd/even) keyed by 0 or 1 and forward the onNext(T) to the matching one.
1. Add one of the pooled dispatchers for the two emitted Stream by previous GroupByAction. Effectively this is scaling up a stream by using 2 partitions 1. assigned with their own dispatcher. FlatMap will merge the result returned by both partitions, running on one of the two threads, but never concurrently.
1. Create 5 Streams and forward onNext(T) to them in a round robin fashion
1. Use the second dispatcher pool of 5 to assign to the newly generated streams. The returned sequences will be merged.
1. Dispatch data on the Environment.sharedDispatcher(), so neither the first or the second pool. The 5 threads will then be merged under the Dispatcher thread

**Output extract**

```
03:53:42.060 [groupByPool-3] INFO  groupBy - onNext: 4
03:53:42.060 [partitionPool-8] INFO  partition - onNext: 9
03:53:42.061 [groupByPool-3] INFO  groupBy - onNext: 6
03:53:42.061 [partitionPool-8] INFO  partition - onNext: 4
03:53:42.061 [shared-1] INFO  join - onNext: 9
03:53:42.061 [groupByPool-3] INFO  groupBy - onNext: 8
03:53:42.061 [partitionPool-4] INFO  partition - onNext: 6
03:53:42.061 [shared-1] INFO  join - onNext: 4
03:53:42.061 [groupByPool-3] INFO  groupBy - onNext: 10
03:53:42.061 [shared-1] INFO  join - onNext: 6
03:53:42.061 [groupByPool-3] INFO  groupBy - complete: DispatcherAction
```

**Table 21. Grouping operations**

|	```	Stream<T> API	```	|	```	Output Type	```	|
|----|----|
|	```	Role	```	|				|
|				|				|
|	```	groupBy(Function<T,K>)	```	|	```	GroupedStream<K,T>	```	|
|				|				|
|	```	partition(int)	```	|	```	GroupedStream<K,T>	```	|
|				|				|
|	```	All window(arguments)	```	|	```	Stream<T>	```	|
|		Windows are actually for cutting partitions over time, size or coordinated with external signals.		|				|
|	```	process(XXXWorkProcessor)	```	|	```	T	```	|
|		Since a RingBufferWorkProcessor distributes the signals to each subscribe, it is an efficient alternative to partition() when its just about scaling-up, not routing.		|				|