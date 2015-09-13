
### Functional Composition

Similar to many other functional libraries, Reactor provides a number of useful methods for composing functions on a Stream. You can passively observe values, transform them from one kind to another, filter out values you don’t want, buffer values until a size or time trigger is tripped, and many other useful operations.

> These operations are called Actions, and they will not wire up the Stream directly. They are available on any Stream instance, which means you should have one by this stage.

* Actions are onSubscribe() in declarative order (left to right), so stream.actionA().actionB() will execute actionA first then actionB.
 * onSubscribe() runs on the parent Publisher thread context which can be altered by subscribeOn(Dispatcher) for instance.
* Actions subscribe() in inverse declarative order (right to left). Whenever subscribe is excplicitely called at the end of the pipeline, subscribe() propagates backward.
 * subscribe() synchronously propagates back which might affect stack size use. If that becomes an issue, use a delegate Processor that runs subscribe() on a Environment.tailRecurse() dispatcher. Then process() it at any point of the chain.

**Observe**

If you want to passively observe data as it passes through the pipeline, then use the .observe(Consumer) methods and other reactor.rx.action.passive actions. To observe values, use .observe(Consumer<? super T>). To observe errors without dealing with them definitively, use .observe(Class<? extends Throwable>, BiConsumer<Object,? extends Throwable>). To observe the Reactive Streams complete signal, use .observeComplete(Consumer<Void>). To observe the cancel signal, use .observeCancel(Consumer<Void>). To observe the Reactive Streams subscribe signal, use .observeSubscribe(Consumer<? super Subscription<T>>).

**observe(Consumer<T>)**

```
Stream<String> st;

st.observe(s -> LOG.info("Got input [{}] on thread [{}}]", s, Thread.currentThread())) (1)
  .observeComplete(v -> LOG.info("Stream is complete")) (2)
  .observeError(Throwable.class, (o, t) -> LOG.error("{} caused an error: {}", o, t)) (3)
  .consume(s -> service.doWork(s)); (4)
```

1. Passively observe values passing through without producing demand.
1. Run once all values have been processed and the Stream is marked complete.
1. Run any time an error is propagated.
1. Produce demand on the pipeline and consume any values.

**Filter**

It’s possible to filter items passing through a Stream so that downstream actions only see the data you want them to see. Filtering actions can be found under the reactor.rx.action.filter package. The most popular one is the .filter(Predicate<T>) method.

> Unmatched data will trigger a Subscription.request(1) if the stream is actually not unbounded with a previous demand of Long.MAX_VALUE.

**filter(Predicate<T>)**

```
Stream<String> st;

st.filter(s -> s.startsWith("Hello"))   (1)
  .consume(s -> service.doWork(s));     (2)
```

1. This will only allow values that start with the string 'Hello' to pass downstream.
1. Produce demand on the pipeline and consume any values.

**Limits**

A specific application of filters is for setting limits to a Stream. Limiting actions can be found under the reactor.rx.action.filter package. There are various ways to tell a Stream<T> its boundary in time, in size and/or on a specific condition. The most popular one is the .take(long) method.

**Stream.take(long)**

```
Streams
  .range(1, 100)
  .take(50)     (1)
  .consume(
    System.out::println,
    Throwable::printStackTrace,
    avoid -> System.out.println("--complete--")
  );
```

1. Only take the 50 first elements then cancel upstream and complete downstream.

**Transformation**

If you want to actively transform data as it passes through the pipeline, then use .map(Function) and other reactor.rx.action.transformation actions. The most popular transforming action is .map(Function<? super I, ? extends O>). A few other Actions depend on transforming data, especially Combinatory operations like flatMap or concatMap.

**Stream.map(Function<T,V>)**

```
Streams
  .range(1, 100)
  .map(number -> ""+number)     (1)
  .consume(System.out::println);
```

1. Transform each Long into a String.

**(A)Sync Transformation: FlatMap, ConcatMap, SwitchMap**

If you want to execute a distinct pipeline Stream<V> or Publisher<V> given an actual input data, you can use combinatory actions such as .flatMap(Function) and other reactor.rx.action.combination actions.

To transform values into a distinct, possibly asynchronous Publisher<V>, use .flatMap(Function<? super I, ? extends Publisher<? extends O>). The returned Publisher<V> will then be merged back to the main flow signaling onNext(V). They are properly removed from the merging action whey they complete. The difference between flatMap, concatMap and switchOnMap is the merging strategy, respectively Interleave, Fully Sequential and Partially Sequential (interrupted by onNext(Publisher<T>)).

> The downstream request is split (minimum 1 by merged Publisher)

**Stream.flatMap(Function)**

```
Streams
  .range(1, 100)
  .flatMap(number -> Streams.range(1, number).subscribeOn(Environment.workDispatcher()) )     (1)
  .consume(
    System.out::println,            (2)
    Throwable::printStackTrace,
    avoid -> System.out.println("--complete--")
  );
```

1. Transform any incoming number into a range of 1-N number merged back and executed on the given Dispatcher.

**Blocking and Promises**

Blocking is considered an anti-pattern in Reactor. That said, we do offer an appropriate API (Ah AH!) for integration with legacy operations and for testing support.

The Promise API offers a range of stateful actions which inspect the current ready|error|complete state and, if fulfilled, immediately calls the wired action.

**Stream.toList()**

```
Promise<List<Long>> result = Streams
  .range(1, 100)
  .subscribeOn(Environment.workDispatcher())
  .toList();    (1)

System.out.println(result.await());     (2)
result.onSuccess(System.out::println);  (3)
```

1. Consume the entire sequence on the dispatcher thread given in subscribeOn(Dispatcher) operation.
1. Block (default 30 Seconds) until onComplete() and print only onNext(List<Long>); or, if onError(e), wrap as RuntimeException and re-raise.
1. Since the promise is already fulfilled, System.out.println() will run immediately on the current context.

**Table 9. Waiting for a Stream or Promise
**

| Functional API or Factory method |
|----|
| Role |
|  |
|  |
| Streams.await(Publisher<?>) |
| Block until the passed Publisher onComplete() or onError(e), bubbling up the eventual exception. |
| Stream.next() |
|  |
| with Promise.await(), Promise.get()… |
| Capture in a Promise the immediate next signal only and onComplete() if the signal was a data. get() can be used to touch but not wait on the promise to fulfill. |
| Stream.toList() |
|  |
| with Promise.await(), Promise.get()… |
| Similar to next() but capture the full sequence in a List<T> to fulfill the Promise<List<T>> returned. |
| Stream.toBlockingQueue() |
| Subscribe to the Stream and return an iterable blocking Queue<T> accumulating all onNext signals.CompletableBlockingQueue.isTerminated() can be used as a condition to exit a blocking poll() loop. |
| Wiring up Synchronous Streams |
| It’s not specific to any API, but if the current Stream is dispatched on a SynchronousDispatcher, it is actually blocking when aterminal action is starting, such as consume(). |