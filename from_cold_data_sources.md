
### From Cold Data Sources

You can create a `Stream` from a variety of sources, including an `Iterable` of known values, a single value to use as the basis for a flow of tasks, or even from blocking structures such as `Future` of `Supplier`.

**Streams.just()**

```
Stream<String> st = Streams.just("Hello ", "World", "!"); (1)

st.dispatchOn(Environment.cachedDispatcher()) (2)
  .map(String::toUpperCase) (3)
  .consume(s -> System.out.printf("%s greeting = %s%n", Thread.currentThread(), s)); (4)
```

1. Create a Stream from a known value but do not assign a default Dispatcher.
1. .dispatchOn(Dispatcher) tells the Stream which thread to execute tasks on. Use this to move execution from one thread to another.
1. Transform the input using a commonly-found convention: the map() method.
1. Produce demand on the pipeline, which means "start processing now". It’s an optimize shortcut for subscribe(Subscriber) where the Subscriber just requests Long.MAX_VALUE by default.

> *Cold Data Sources will be replayed from start for every fresh Subscriber passed to `Stream.subscribe(Subscriber)`, and therefore duplicate consuming is possible.*

**Table 5. Creating pre-determined Streams and Promises**


