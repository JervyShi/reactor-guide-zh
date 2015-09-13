
### Into Windows

Forwarding grouped sequences of data T into a Stream<T> serves three main purposes:

* Expose a sequence of data T to various limited grouped observations and accumulation: metrics, average, flexible aggregate (Map, Tuple…​).
* Parallelizing grouped sequences combined with dispatchOn for each generated Stream<T> and merging their results back.
* Repeat onComplete() for individual grouped sequences, e.g. in Async IO module to delimit a flush.

> Stream<T> windows are slightly less optimized but equivalent aggregating producer than buffer API if combined with the aggregate-all Stream.buffer() method:

```
stream.buffer(10, 1, TimeUnit.SECONDS);

//equivalent to
stream.window(10, 1, TimeUnit.SECONDS).flatMap( window -> window.buffer() )
```
> An Environment must be initialized if the alias for timed window() are used without providing the Timer argument.

```
//create a list of 1000 numbers and prepare a Stream to read it
Stream<Integer> sensorDataStream = Streams.from(createTestDataset(1000));

//wait for all windows of 100 to finish
CountDownLatch endLatch = new CountDownLatch(1000 / 100);

Control controls = sensorDataStream
  .window(100)
  .consume(window -> {
    System.out.println("New window starting");
    window
      .reduce(Integer.MAX_VALUE, (acc, next) -> Math.min(acc, next))
      .finallyDo(o -> endLatch.countDown())
      .consume(i -> System.out.println("Minimum " + i));
  });

endLatch.await(10, TimeUnit.SECONDS);
System.out.println(controls.debug());

Assert.assertEquals(0, endLatch.getCount());
```

**Table 11. Chunk processing with Stream (returning Stream<Stream<T>>):**

|	Stream<T> API	|
|---------------|
|	Role	|
|		|
|	window(int)	|
|	Forward to a generated Stream<T> until onComplete() or the given int argument is reached which starts over a new Stream.	|
|	window(Publisher<?>, Supplier<? extends Publisher<?>>)	|
|	Forward to a generated Stream<T> until onComplete() or when the first Publisher<?> argument emits a signal. The optionalSupplier<? extends Publisher<?>> supplies a sequence whose first signal will end the linked aggregation. That means overlapping (sliding buffers) and disjointed aggregations can be emitted to the child Subscriber<Stream<T>>.	|
|	window(Supplier<? extends Publisher<?>>)	|
|	Forward to a generated Stream<T> until onComplete() or in coordination with a provided Publisher<?>. TheSupplier<? extends Publisher<?>> supplies a sequence whose first signal will end the linked Stream<T> and start a new one immediately.	|
|	window(int, int)	|
|	Forward to a generated Stream<T> until onComplete() or the given skip (the second int argument) is reached which starts over a new Stream<T>. The size (the first int argument) will delimit the maximum numger of aggregated elements by buffer. That means overlapping (sliding buffers) and disjointed sequences can be emitted to the child Subscriber<Stream<T>>.	|
|	window(long, TimeUnit, Timer_)	|
|	Forward to a generated Stream<T> until onComplete() or the elpased period (the long argument) is reached, which starts over a new Stream<T>.	|
|	window(long, long, TimeUnit, Timer_)	|
|	Forward to a generated Stream<T> until onComplete() or the given timeshift (the second long argument) is reached. The timespan(the first long argument) will delimit the maximum numger of aggregated elements by buffer. That means overlapping (sliding buffers) and disjointed sequenced can be emitted to the child Subscriber<Stream<T>>.	|
|	window(int, long, TimeUnit, Timer)	|
|	A combination of buffer(int) OR buffer(long, TimeUnit, Timer) conditons. It forwards to a generated Stream<T> until the given size has been reached or the timespan has elapsed.	|