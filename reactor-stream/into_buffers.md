
### Into Buffers

收集的数据分组序列`T`到集合`List<T>`主要有两个目的：

* 使用JVM API常用的可迭代结构来暴露匹配边界条件的序列。
* 减少`onNext(T)`信号的数量，例如：`buffer(5)`将会把包含10个元素的序列转换为2个包含5个元素的集合。

> 组合数据引发本应当适量大小的内存和CPU过载。小巧和定时边界明智的避免长时间的聚集。

> 如果定时`buffer()`信号没有提供定时参数，一个环境必须先被初始化。

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