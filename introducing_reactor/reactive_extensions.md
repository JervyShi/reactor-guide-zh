# Reactive Extensions

Reactive Extensions, or more commonly Rx, are a set of well-defined Functional APIs extending the Observer pattern to an epic scale.

**Rx patterns support implementing Reactive data sequences handling with a few design keys:**

* Abstract the time/latency away with a callback chain: only called when data is available
* Abstract the threading model away: Synchronous or Asynchronous it is just an Observable / Stream we deal with
* Control error-passing and terminations: error and complete signals in addition to the data payload signal are passed to the chain
* Solve multiple scatter-aggregation and other composition issues in various predefined API.

The standard implementation of Reactive Extensions in the JVM is RxJava. It provides a powerful functional API and ports mostly all the concept over from the original Microsoft library.

Reactor 2 provides a specific module implementing a subset of the documented Reactive Extensions and on a very few occasion adapting the name to match our specific behavior. This focused approach around data-centric issues (microbatching, composition…) is depending on Reactor Functional units, Dispatchers and the Reactive Streams contract. We encourage users who need the full flavor of Reactive Extensions to try out RxJava and bridge with us. In the end the user can benefit from powerful asynchronous and IO capacities provided by Reactor while composing with the complete RxJava ecosystem.

> Some operations, behaviors, and the immediate understanding of Reactive Streams are still unique to Reactor as of now and we will try to flesh out the unique features in the appropriate section.

> Async IO capabilities are also depending on Stream Capacity for backpressure and auto-flush options.

**Table 2. Misalignments between Rx and Reactor Streams**

rx|reactor-stream|Comment
--|--------------|-------
Observable|reactor.rx.Stream|Reflect the implementation of the Reactive Stream Publisher
Operator|reactor.rx.action.Action|Reflect the implementation of the Reactive Stream Processor
Observable with 1 data at most|reactor.rx.Promise|Type a unique result, reflect the implementation of the Reactive Stream Processor and provides for optional asynchronous dispatching.
Factory API (just, from, merge….)|reactor.rx.Streams|Aligned with a core data-focused subset, return Stream
Functional API (map, filter, take….)|reactor.rx.Stream|Aligned with a core data-focused subset, return Stream
Schedulers|reactor.core.Dispatcher, org.reactivestreams.Processor|Reactor Streams compute operations with unbounded shared Dispatchers or bounded Processors
Observable.observeOn()|Stream.dispatchOn()|Just an adapted naming for the dispatcher argument


