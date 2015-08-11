# Core Processors
Core Processors are here to do a more focused job than Dispatchers: Computing asynchronous tasks with back-pressure support.

They also play nicely with other Reactive Streams vendors since they directly implement the org.reactivestreams.Processor interface. Remember that a Processor is both a Subscriber AND a Publisher, so you can insert it in a Reactive Streams chain where you wish (source, processing, sink).

> The specification doesn’t recommend specifically to hit Processor.onNext(d) directly. We do offer that support but the backpressure will of course not be propagated except with eventual blocking. One can explicitely use an anonymous Subscription to pass first to a Processor using Processor.onSubscribe to get the backpressure feedback within the implemented request method.

> OnNext must be serialized e.g. coming from a single thread at a time (no concurrent onXXX signal is allowed). However Reactor supports it if the Processors are created using the conventioned Processor.share() method, e.g. RingBufferProcessor.share(). This decision must be taken at creation time in order to use the right coordination logic within the implementation, so choose wisely: is this going to be a a standard publishing sequence (no concurrent) or is this going to be hit by multiple threads ?

***Reactor makes a single exception to the standard when it comes to the specific XXXX Work Processor artefacts:***

* Usually Reactive Streams Processor will dispatch the same sequence data asynchronously to all Subscribers subscribed at a given time T. It’s akin to **Publish/Subscribe** pattern.
* **WorkProcessors** will distribute the data to its convenience, making the most of each Subscriber. That means Subscribers at a given time T will always see distinct data. It’s akin to **WorkQueue** pattern.

We expect to increase our collection of Core Processors over the 2.x release line.
