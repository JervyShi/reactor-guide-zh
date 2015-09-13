**Into Buffers**

Collecting grouped sequences of data T into lists List<T> serves two main purposes:

* Expose a sequence matching the boundary conditions into an Iterable structure commonly used by JVM APIs
* Reduce the volume of onNext(T) signals, e.g. buffer(5) will transform a sequence of 10 elements into a sequence of 2 lists (of 5 elements).

> Collecting data incurs an overhead in memory and possibly CPU that should be sized appropriately. Small and timed boundaries are advised to avoid any long lasting aggregates.

> An Environment must be initialized if the timed buffer() signatures are used without providing the Timer argument.

```
long timeout = 100;
final int batchsize = 4;
CountDownLatch latch = new CountDownLatch(1);

final Broadcaster<Integer> streamBatcher = Broadcaster.<Integer>create(env);
streamBatcher
  .buffer(batchsize, timeout, TimeUnit.MILLISECONDS)
  .consume(i -> latch.countDown());


streamBatcher.onNext(12);
streamBatcher.onNext(123);
Thread.sleep(200);
streamBatcher.onNext(42);
streamBatcher.onNext(666);

latch.await(2, TimeUnit.SECONDS);
```

**Table 10. Chunk processing with Stream buffers (returning Stream<List<T>>):
**

| Stream<T> API |
|---------------|
|	Role	|
|		|
|	buffer(int)	|
|	Aggregate until onComplete() or the given int argument is reached which starts over a new aggregation.	|
|	buffer(Publisher<?>, Supplier<? extends Publisher<?>>)	|
|	Aggregate until onComplete() or when the first Publisher<?> argument emits a signal. The optionalSupplier<? extends Publisher<?>> supplies a sequence whose first signal will end the linked aggregation. That means overlapping (sliding buffers) and disjointed aggregation can be emitted to the child Subscriber<List<T>>.	|
|	buffer(Supplier<? extends Publisher<?>>)	|
|	Aggregate until onComplete() or in coordination with a provided Publisher<?>. The Supplier<? extends Publisher<?>>supplies a sequence whose first signal will end the linked aggregation and start a new one immediately.	|
|	buffer(int, int)	|
|	Aggregate until onComplete() or the given skip (the second int argument) is reached which starts over a new aggregation. The firstsize int argument will delimit the maximum numger of aggregated elements by buffer. That means overlapping (sliding buffers) and disjointed aggregation can be emitted to the child Subscriber<List<T>>.	|
|	buffer(long, TimeUnit, Timer_)	|
|	Aggregate until onComplete() or the elapsed period (the first long argument) is reached, which starts over a new aggregation.	|
|	buffer(long, long, TimeUnit, Timer_)	|
|	Aggregate until onComplete() or the given timeshift (the second long argument) is reached. The timespan (the first long argument) will delimit the maximum numger of aggregated elements by buffer. That means overlapping (sliding buffers) and disjointed aggregation can be emitted to the child Subscriber<List<T>>.	|
|	buffer(int, long, TimeUnit, Timer)	|
|	A combination of buffer(int) OR buffer(long, TimeUnit, Timer) conditons. It accumulates until the given size has been reached or the timespan has elapsed.	|