
## Analytics

Metrics and other stateful operations are fully part of the Stream API. Users familiar with Spark will recognize some method names in fact. ScanAction also offers a popular accumulating functional contract with reduce() and scan().

**Playing with metrics and key/value data**

```
Broadcaster<Integer> source = Broadcaster.<Integer> create(Environment.get());
long avgTime = 50l;

Promise<Long> result = source
    .throttle(avgTime)  (1)
    .elapsed()  (2)
    .nest()     (3)
    .flatMap(self ->
            BiStreams.reduceByKey(self, (prev, next) -> prev + 1)    (4)
    )
    .sort((a,b) -> a.t1.compareTo(b.t1))    (5)
    .log("elapsed")
    .reduce(-1L, (acc, next) ->
            acc > 0l ? ((next.t1 + acc) / 2) : next.t1  (6)
    )
    .next();    (7)

for (int i = 0; i < 10; i++) {
  source.onNext(1);
}
source.onComplete();
```

1. Slow down incoming Subscriber request to one every ~50 milliseconds, polling waiting data one by one.
1. Produce a Tuple2 of Time delta and payload between 2 signals or between onSubscribe and the first signal.
1. Make the current Stream available with onNext so we can compose it with a flatMap.
1. Accumulate all data until onComplete() in internal Map keyed with the Tuple2.t1 and valued by default with Tuple2.t2. Next matching keys will provide 1. the previous value and the incoming new onNext in the accumulator BiFunction. In this case we only increment the initial payload 1 by key.
1. Accumulate all data until onComplete() in internal PriorityQueue and sort elapsed time t1 using the given comparator. After onComplete() all data are 1. emitted in order, then complete.
1. Accumulate until onComplete a moving time average defaulting to the first received time.
1. Take the next and only produced average.

**Output**

```
03:14:42.013 [main] INFO  elapsed - subscribe: ScanAction
03:14:42.021 [main] INFO  elapsed - onSubscribe: {push}
03:14:42.022 [main] INFO  elapsed - request: 9223372036854775807
03:14:42.517 [hash-wheel-timer-run-3] INFO  elapsed - onNext: 44,1
03:14:42.518 [hash-wheel-timer-run-3] INFO  elapsed - onNext: 48,1
03:14:42.518 [hash-wheel-timer-run-3] INFO  elapsed - onNext: 49,2
03:14:42.518 [hash-wheel-timer-run-3] INFO  elapsed - onNext: 50,3
03:14:42.518 [hash-wheel-timer-run-3] INFO  elapsed - onNext: 51,3
03:14:42.519 [hash-wheel-timer-run-3] INFO  elapsed - complete: SortAction
03:14:42.520 [hash-wheel-timer-run-3] INFO  elapsed - cancel: SortAction
```

**Table 20. Operations useful for metrics and other stateful accumulation.**

|	```Stream<T>``` API or Factory method	|	Output Type	|
|----|----|
|	Role	|		|
|		|		|
|	count()	|	Long	|
|	Produce the total number of observed onNext(T) after observing onComplete(). Useful when combined with timed windows. Not so useful with sized windows, e.g. stream.window(5).flatMap(w -> w.count()) → produce 5, awesome.	|		|
|	```scan(BiFunction<T,T>)```	|	T	|
|		|		|
|	```scan(A, BiFunction<A,T>)```	|	A	|
|		|		|
|	```reduce(BiFunction<T,T>)```	|	T	|
|		|		|
|	```reduce(A, BiFunction<A,T>)```	|	A	|
|		|		|
|	```BiStreams.reduceByKey()```	|		|
|		|		|
|	```BiStreams.scanByKey()```	|		|
|		|		|
|	```timestamp()```	|	```Tuple2<Long,T>```	|
|		|		|
|	```elapsed()```	|	```Tuple2<Long,T>```	|
|		|		|
|	```materialize() dematerialize()```	|	```Signal<T>```	|
|	Transform upstream signal into ```Signal<T>```, and treat them as ```onNext(Signal<T>)``` signals. The immediate effect: it swallows error and completion signals, so it’s an effective way to count errors and completions if the Stream is using retry or repeat API. Once completion and errors are processed we can still run them by transforming the ```Signal<T>``` into the Reactive Streams right callback ```viadematerialize()```.	|		|