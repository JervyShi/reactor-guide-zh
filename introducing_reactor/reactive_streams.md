# Reactive Streams

Reactive Streams is a new standard, adopted by different vendors and tech industrials including Netflix, Oracle, Pivotal or Typesafe with a target to include the specification into Java 9 and onwards.

The aim of the standard is to provide (a)synchronous data sequences with a flow-control mechanism. The specification is fairly light and first targets the JVM. It comes with 4 Java Interfaces, a TCK and a handful of examples. It is quite straightforward to implement the 4 interfaces for the need, but the meat of the project is actually the behaviors verified by the TCK. A provider is qualified Reactive Streams Ready since it successfully passed the TCK for the implementing classes, which fortunately we did.

![Figure 3. The Reactive Streams Contract](http://projectreactor.io/docs/reference/images/rs.png)
**Figure 3. The Reactive Streams Contract**

**The Reactive Streams Interfaces**

* org.reactivestreams.Pubslisher: A source of data (from 0 to N signals where N can be unlimited). It optionally provides for 2 terminal events: error and completion.
* org.reactivestreams.Subscriber: A consumer of a data sequence (from 0 to N signals where N can be unlimited). It receives a subscription on initialization to request how many data it wants to process next. The other callbacks interact with the data sequence signals: next (new message) and the optional completion/error.
* org.reactivestreams.Subscription: A small tracker passed on initialization to the Subscriber. It controls how many data we are ready to consume and when do we want to stop consuming (cancel).
* org.reactivestreams.Processor: A marker for components that are both Subscriber and Publisher!

![Figure 4. The Reactive Streams publishing protocol
](http://projectreactor.io/docs/reference/images/signals.png)
**Figure 4. The Reactive Streams publishing protocol**

**There are two ways to request data to a Publisher from a Subscriber, through the passed Subscription:**

* Unbounded: On Subscribe, just call Subscription#request(Long.MAX_VALUE).
* Bounded: On Subscribe, keep a reference to Subscription and hit its request(long) method when the Subscriber is ready to process data.
 * Typically, Subscribers will request an initial set of data, or even 1 data on Subscribe
 * Then after onNext has been deemed successful (e.g. after Commit, Flush etc…), request more data
 * It is encouraged to use a linear number of requests. Try avoiding overlapping requests, e.g. requesting 10 more data every next signal.


Table 1. What are the artifacts that Reactor directly use so far:

Reactive Streams|Reactor Module(s)|Implementation(s)|Description
----------------|-----------------|-----------------|-----------
Processor|reactor-core, reactor-stream|reactor.core.processor.*, reactor.rx.*|In Core, we offer backpressure-ready RingBuffer*Processor and more, in Stream we have a full set of Operations and Broadcasters.
Publisher|reactor-core, reactor-bus, reactor-stream, reactor-net|reactor.core.processor.*, reactor.rx.stream.*, reactor.rx.action.*, reactor.io.net.*|In Core, processors implement Publisher. In Bus we publish an unbounded emission of routed events. In Stream, our Stream extensions directly implement Publisher. In Net, Channels implement Publisher to consume incoming data, we also provide publishers for flush and close callbacks.
Subscriber|reactor-core, reactor-bus, reactor-stream, reactor-net|reactor.core.processor.*, reactor.bus.EventBus.*, reactor.rx.action.*, reactor.io.net.impl.*|In Core, our processor implement Subscriber. In Bus, we expose bus capacities with unbounded Publisher/Subscriber. In Stream, actions are Subscribers computing specific callbacks. In Net, our IO layer implements subscribers to handle writes, closes and flushes.
Subscription|reactor-stream, reactor-net|reactor.rx.subscription.*, reactor.io.net.impl.*|In Stream, we offer optimized PushSubscriptions and buffering-ready ReactiveSubscription. In Net, our Async IO reader-side use custom Subscriptions to implement backpressure.

We have worked with the standard since the inception of Reactor 2 and progressed in our journey until the 1.0.0 was about to release. It is now available on Maven Central and other popular mirrors. You will also find it as a transitive dependency to reactor-core.

