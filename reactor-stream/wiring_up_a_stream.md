
### Wiring up a Stream

Streams operations—except for a few exceptions like terminal actions and broadcast()—will never directly subscribe. Instead they will lazily prepare for subscribe. This is usually called lift in Functional programming.

That basically means the Reactor Stream user will explicitely call Stream.subscribe(Subscriber) or, alternativly, terminal actions such as Stream.consume(Consumer) to materialize all the registered operations. Before that Actions don’t really exist. We use Stream.lift(Supplier) to defer the creation of these Actions until Stream.subscribe(Subscriber) is explicitely called.

Once everything is wired, each action maintains an upstream Subscription and a downstream Subscription and the Reactive Streams contract applies all along the pipeline.

> Usually the terminal actions return a Control object instead of Stream. This is an component you can use to request or cancel a pipeline without being inside a Subscriber context or implementing the full Subscriber contract.

**Wiring up 2 pipelines**

```
import static reactor.Environment.*;
import reactor.rx.Streams;
import reactor.rx.Stream;
//...

Stream<String> stream = Streams.just("a","b","c","d","e","f","g","h");

//prepare two unique pipelines
Stream<String> actionChain1 = stream.map(String::toUpperCase).filter(w -> w.equals("C"));
Stream<Long> actionChain2 = stream.dispatchOn(sharedDispatcher()).take(5).count();

actionChain1.consume(System.out::println); //start chain1
Control c = actionChain2.consume(System.out::println); //start chain2
//...
c.cancel(); //force this consumer to stop receiving data
```

![Figure 10. After Wiring](http://projectreactor.io/docs/reference/images/wiringup.png)
**Figure 10. After Wiring**

Publish/Subscribe

For Fan-Out to subscribers from a unified pipeline, Stream.process(Processor), Stream.broadcast(), Stream.broadcastOn() and Stream.broadcastTo() can be used.

Sharing an upstream pipeline and wiring up 2 downstream pipelines

```
import static reactor.Environment.*;
import reactor.rx.Streams;
import reactor.rx.Stream;
//...

Stream<String> stream = Streams.just("a","b","c","d","e","f","g","h");

//prepare a shared pipeline
Stream<String> sharedStream = stream.observe(System.out::println).broadcast();

//prepare two unique pipelines
Stream<String> actionChain1 = sharedStream.map(String::toUpperCase).filter(w -> w.equals("C"));
Stream<Long> actionChain2 = sharedStream.take(5).count();

actionChain1.consume(System.out::println); //start chain1
actionChain2.consume(System.out::println); //start chain2
```

![Figure 11. After Wiring a Shared Stream](http://projectreactor.io/docs/reference/images/broadcast.png)
**Figure 11. After Wiring a Shared Stream**

**Table 8. Operations considered terminal or explicitely subscribing
**

| Stream<T> method  | Return Type |
|-------------------|-------------|
| Role  |   |
| subscribe(Subscriber<T>)  | void  |
| subscribeOn |   |
| Subscribe the passed Subscriber<T> and materialize any pending upstream, wired up lazily (the implicit lift for non terminal operation). Note a Subscriber must request data if it expects some. The dispatchOn and subscribeOn alternatives provide for signallingonSubscribe using the passed Dispatcher. |   |
| consume(Consumer<T>,Consumer<T>,Consumer<T>)  | Control |
| consumeOn |   |
| Call subscribe with a ConsumerAction which interacts with each passed Consumer, each time the interest signal is detected. It willrequest(Streams.capacity()) to the received Subscription, which is Long.MAX_VALUE by default, which results in unbounded consuming. The subscribeOn and consumeOn alternatives provide for signalling onSubscribe using the passedDispatcher. Returns a Control component to cancel the materialized Stream, if necessary. Note that ConsumeAction takes care of unbounded recursion if the onNext(T) signal triggers a blocking request. |   |
| consumeLater()  | Control |
| Similar to consume but does not fire an initial Subscription.request(long). The returned Control can be used torequest(long) anytime. |   |
| tap() | TapAndControls  |
| Similar to consume but returns a TapAndControls that will be dynamically updated each time a new onNext(T) is signalled or cancelled. |   |
| batchConsume(Consumer<T>, Consumer<T>, Consumer<T>, Function<Long,Long>)  | Control |
| batchConsumeOn  |   |
| Similar to consume but will request the mapped Long demand given the previous demand and starting with the defaultStream.capacity(). Useful for adapting the demand dynamically due to various factors. |   |
| adaptiveConsume(Consumer<T>, Consumer<T>, Consumer<T>,Function<Stream<Long>,Publisher<Long>>) | Control |
| adaptiveConsumeOn |   |
| Similar to batchConsume but will request the computed sequence of demand Long. It can be used to insert flow-control such asStreams.timer() to delay demand.  |   |
| next()  | Promise<T>  |
| Return a Promise<T> that is actively subscribing to the Stream, materializing it, and requesting a single data before unregistering. The immediate next signal onNext(T), onComplete() or onError(Throwable) will fulfill the promise.  |   |
| toList()  | Promise<List<T>>  |
| Similar to next() but will wait until the entire sequence has been produced (onComplete()) and pass the accumulated onNext(T) in a single List<T> fulfilling the returned promise.  |   |
| Stream.toBlockingQueue()  | CompletableBlockingQueue<T> |
| Subscribe to the Stream and return an iterable blocking Queue<T> accumulating all onNext signals.CompletableBlockingQueue.isTerminated() can be used as a condition to exit a blocking poll() loop. |   |
| cache() | Stream<T> |
| Turn any Stream into a Cold Stream, able to replay all the sequence of signals individually for each Subscriber. Due to the unbounded nature of the action, you should probably use it only with small(ish) sequences.  |   |
| broadcast() | Stream<T> |
| broadcastOn(Environment, Dispatcher)  |   |
| Turn Any Stream into a Hot Stream. This will prevent pipeline duplication by immediately materializing the Stream and be ready to publish the signal to N Subscribers downstream. The demand will be aggregated from all child Subscribers. |   |
| broadcastTo(Subscriber<T>)  | Subscriber<T> |
| An alternative to Stream.subscribe which allows method chaining since the returned instance is the same as the passed argument. |   |
| process(Processor<T, O>)  | Stream<O> |
| Similar to broadcast() but accept any given Processor<T, O>. A perfect place to introduce Core Processors ! |   |