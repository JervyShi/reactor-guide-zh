
## Streams Basics

Reactor offers `Stream` or `Promise` based on the Reactive Streams standard to compose statically typed data pipelines.

It is an incredibly useful and flexible component. It’s flexible enough to be used to just compose asynchronous actions together like RxJava’s `Observable`. But it’s powerful enough it can function as an asynchronous work queue that forks and joins arbitrary compositions or other Reactive Streams components coming from one of the other implementors of the standard.[3].

**There are basically two rough categories of streams**

* A hot Stream is unbounded and capable of accepting input data like a sink.
 * Think UI events such as mouse clicks or realtime feeds such as sensors, trade positions or Twitter.
 * Adapted backpressure strategies mixed with the Reactive Streams protocol will apply
* A cold Stream is bounded and generally created from a fixed collection of data like a List or other Iterable.
 * Think Cursored Read such as IO reads, database queries,
 * Automatic Reactive Streams backpressure will apply

> *As seen previously, Reactor uses an Environment to keep sets of Dispatcher instances around for shared use in a given JVM (and classloader). Anstatic {
  Environment.initialize();
} Environment instance can be created and passed around in an application to avoid classloading segregation issues or the static helpers can be used. Throughout the examples on this site, we’ll use the static helpers and encourage you to do likewise. To do that, you’ll need to initialize the static Environment somewhere in your application.*

> ```static {
  Environment.initialize();
}```