
### Wiring up a Stream

Streams operations — except for a few exceptions like terminal actions and broadcast() — will never directly subscribe. Instead they will lazily prepare for subscribe. This is usually called lift in Functional programming.

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

