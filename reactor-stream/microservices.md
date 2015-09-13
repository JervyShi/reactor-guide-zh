
## MicroServices

The notion of MicroService has been an increasingly popular term over the last years. Simply put, we code software components with a focused purpose to encourage isolation, adapted scaling and reuse. In fact it has been over 30 years we use them:

**An example of microservices in Unix**

```history | grep password```

Even within the boundaries of the application, we can find the similar concept of functional granularity:

**An example of microservices in imperative Java code**

```
User rick = userService.get("Rick");
User morty = userService.get("Morty");
List<Mission> assigned = missionService.findAllByUserAndUser(rick, morty);
```

Of course the application has been widely popular within distributed systems and cloud-ready architectures. When the function is isolated enough, it will depend on N other ones for : Data access, subroutine calls over network, posting into message bus, querying an HTTP REST endpoint etc. This is where troubles begin: the execution flow is crossing multiple context boundaries. Relatively latency and failure will start to scale up as the system grows in volume and access.

At this point we can decide to scale-out, after all platforms such as CloudFoundry allow for elastic scaling of JVM apps and beyond. But looking at our CPU and memory use, it didn’t seem particularly under pressure. Of course it was not, each remote call was just blocking the whole service and preventing concurrent user requests to kick in: They are just parked in some thread pool queue. In the meantime the active request was happily seating for a few milliseconds or more waiting for a remote HTTP call socket to actually write, a delay we call latency here.

The same applies to errors, we can make applications more resilient (fallbacks, timeouts, retries…) individually first and not rely on scaling out. The classic hope is that a replicate microservice will pick up the requests when a load-balancer will detect the failure:

```
Load Balancer: "are you dead ?"
30 sec later
Load Balancer: "are you dead ?"
30 sec later
Load Balancer: "you're dead !"
MicroService "I'am alive !"
```

In a Distributed System, coordination pulls a very long string of issues you wish you have never faced.

A Publisher like a Stream or a Promise is ideal to confront MicroServices latency and errors. To improve the situation with better error isolation and non-blocking service calls, code has to be designed with these two constraints in mind. To put on your side the best chances for a successful migration story to a Reactive Architecture, you might prefer to work step by step with quick wins and a few adjustements, test and iterate to the next step.

In this section we’re going to cover the basics to create a reactive facade gating each costly remote call, build functional services and make them latency-ready.

**Becoming Reactive with Reactor in 3 steps:**

1. Transform target service calls into unbounded Stream or Promise return types
 * Asynchronous switch for Blocking → Non Blocking conversion

 * Error isolation
1. Compose services with the Reactor Stream API

 * Blocking → Non Blocking coordination

 * Parallelize Blocking calls
1. Evolve transformed services to backpressure ready Stream

 * Chunk processing/reading with bounded access

 * Optimize IO operations with Microbatching

**Table 14. Common Actions at play when reading remote resources**

|	Functional API or Factory method	|
|----|
|	Role	|
|		|
|	Streams.create(Publisher), Streams.defer(Supplier), Streams.wrap(Publisher), Streams.generate(Supplier)	|
|	Protecting resource access with a Publisher is encouraged. A few Stream factories will be particularly useful. The point of creating aPublisher is to only onNext(T) when the data is ready such as in an IO callback. The read should be triggered by a Subscriberrequest if possible to implement a form of backpressure.	|
|	Stream.timeout(arguments)	|
|	Accessing an external resource, especially remote, should always be limited in time to become more resilient to external conditions such as network partitions. Timeout operations can fallback to another Publisher for alternative service call or justonError(TimeoutException). The timer resets each time a fresh onNext(T) is observed.	|
|	Stream.take(arguments)	|
|	Similar to timeout(), a need to scope in size an external resource is a common one. It’s also useful to fully trigger a pipeline includingonComplete() processing.	|
|	Stream.flatMap(Function<T,Publisher<V>)	|
|	An Async transformation that produces a Publisher<V> eventully using the passed data T, the ideal place to hook in a call to another service before resuming the current processing.	|
|		|
|	The sequence of produced Publisher<V> will flow in the Subscriber into a sequence of V by safely subscribing in parallel.	|
|	Stream.subscribeOn(Dispatcher), Stream.dispatchOn(Dispatcher), Core Processors	|
|	Threading control is strategic:	|
|		|
|	* Slow MicroService, low volume or low throughput, e.g. HTTP GET → subscribeOn(workQueueDispatcher()) to scale-up concurrent service calls.	|
|		|
|	* Fast MicroService, high volume or high throughput, e.g. Message Bus → dispatchOn(sharedDispatcher()) orRingBufferXXXProcessor.create() to scale up message-dispatching.	|