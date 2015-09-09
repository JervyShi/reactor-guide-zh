
## From Custom Reactive Publishers

Over time, the Reactor user will become more familiar with Reactive Streams. That’s the perfect moment to start creating custom reactive data-sources! Usually the implementor would have to respect the specification and verify his work with the reactive-streams-tck dependency. Respecting the contract requires a Subscription and a call to onSubscribe + a request(long) before sending any data.

However Reactor allows some flexibility to only deal with the message passing part and will automatically provide the buffering Subscription transparently, the difference is demonstrated in the code sample below.

**Streams.create and Streams.defer in action**

```
final Stream<String> stream1 = Streams.create(new Publisher<String>() {
  @Override
  public void subscribe(Subscriber<? super String> sub) {
    sub.onSubscribe(new Subscription() { (1)
      @Override
      public void request(long demand) {
        if(demand == 2L){
          sub.onNext("1");
          sub.onNext("2");
          sub.onComplete();
        }
      }

      @Override
      public void cancel() {
        System.out.println("Cancelled!");
      }
    });
  }
});

final Stream<String> stream2 = Streams.create(sub -> {
  sub.onNext("3"); (2)
  sub.onNext("4");
  sub.onComplete();
});

final AtomicInteger counterSubscriber = new AtomicInteger();

Stream<String> deferred = Streams.defer(() -> {
  if (counterSubscriber.incrementAndGet() == 1) { (3)
    return stream1;
  }
  else {
     return stream2;
  }
});

deferred
  .consume(s -> System.out.printf("%s First subscription = %s%n", Thread.currentThread(), s));
deferred
  .consume(s -> System.out.printf("%s Second subscription = %s%n", Thread.currentThread(), s));
```

1. Create a Stream from a custom valid Publisher which first calls onSubscribe(Subscription).
1. Create a Stream from a custom malformed Publisher which skips `onSubscribe(Subscription) and immediately calls onNext(T).
1. Create a DeferredStream that will alternate source Publisher<T> on each Stream.subscribe call, evaluating the total number of Subscribers,

Where to go from here? There are plenty of use cases that can benefit from a custom Publisher:

* Reactive Facade to convert any IO call with a matching demand and compose: HTTP calls (read N times), SQL queries (select max N), File reads (read N lines)…
* Async Facade to convert any hot data callback into a composable API: AMQP Consumer, Spring MessageChannel endpoint…

Reactor offers some reusable components to avoid the boilerplate checking you would have to do without extending exsiting Stream or PushSubscription

* Extending PushSubscription instead of implementing Subscription directly to benefit from terminal state (PushSubscription.isComplete())
* Using PublisherFactory.create(args) or Streams.createWith(args) to use Functional consumers for every lifecycle step (requested, stopped, started).
* Extending Stream instead of implementing Publisher directly to benefit from composition API

**Streams.createWith, an alternative to create() minus some boilerplate**

```
final Stream<String> stream = Streams.createWith(
  (demand, sub) -> { (1)
      sub.context(); (2)
      if (demand >= 2L && !sub.isCancelled()) {
          sub.onNext("1");
          sub.onNext("2");
          sub.onComplete();
      }
  },
  sub -> 0, (3)
  sub -> System.out.println("Cancelled!") (4)
);

stream.consume(s -> System.out.printf("%s greeting = %s%n", Thread.currentThread(), s));
```

1. Attach a request consumer reacting on Subscriber requests and passing the demand and the requesting subscriber.
1. The sub argument is actually a SubscriberWithContext possibly assigned with some initial state shared by all request callbacks.
1. Executed once on start, this is also where we initialize the optional shared context; every request callback will receive 0 from context()
1. Executed once on any terminal event : cancel(), onComplete() or onError(e).

A good place to start coding the reactive streams way is to simply look at a more elaborate, back-pressure ready File Stream.