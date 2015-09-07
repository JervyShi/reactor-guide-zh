
## Creating Streams and Promises

This is where you start if you are the owner of the data-source and want to just make it Reactive with direct access to various Reactive Extensions and Reactive Streams capacities.

Sometimes it’s also a case for expanding an existing `Reactive Stream Publisher` with `Stream` API and we fortunately offer one-shot static API to proceed to the conversion.

Extending existing `Reactor Stream` like we do with `IterableStream`, `SingleValueStream` etc is also an incentive option to create a `Publisher` ready source (Stream implements it) injected with Reactor API.

> *Streams and Promises are relatively inexpensive, our microbenchmark suite succeeds into creating more than 150M/s on commodity hardware. Most of the Streams stick to the Share-Nothing pattern, only creating new immutable objects when required.*

> *Every operation will return a new instance:*

```
Stream<A> stream = Streams.just(a);
Stream<B> transformedStream = stream.map(transformationToB);

Assert.isTrue(transformationStream != stream);
stream.subscribe(subscriber1); //subscriber1 will see the data A unaltered
transformedStream.subscribe(subscriber2); //subscriber2 will see the data B after transformation from A.

//Note theat these two subscribers will materialize independant stream pipelines, a process we also call lifting
```
