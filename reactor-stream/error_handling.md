
## Error Handling

Since error isolation is an important part of the Reactive contract, Stream API is equipped to build fault tolerant pipelines or service call.

Error isolation comes simply by preventing onNext, onSubscribe and onComplete callbacks to bubble up any exception. Instead they are passed to the onError callback and propagated downstream. A few Action can react passively or actively on such signal, e.g. when() will just observe errors and onErrorResumeNext() will switch to a fallback Publisher.

Inversing the propagation to the consuming side instead of bubbling up to the producer side is the reactive pattern to isolate the data producer from the pipeline errors and keep producers alive and happy.

In the end the last Subscriber in the chain will be notified with the onError(t) callback method. If that Subscriber is a ConsumerAction for instance, Reactor will re-route an error if no errorConsumer callback has been assigned using Stream.consume(dataConsumer, errorConsumer). The route will trigger the current Environment error journal if set, which by default uses SLF4J to log errors.

**Reactor also distinguishes fatal exceptions from normal ones, specially during onSubscribe process. These exceptions will not be isolated nor passed downstream to the subscriber(s) :**

* CancelException
 * Happens if no subscriber is available during onNext propagation, e.g. when a subscriber asynchronously cancelled during onNext emission
 * Use the JVM property -Dreactor.trace.cancel=true to enable verbose CancelException and logging in Environment default journal. If not set, Environment  * will not report these exceptions and there won’t be any stacktrace associated neither.
* ReactorFatalException
 * Happens when Reactor defines an unrecoverable situation like a scheduling on Timer not matching the resolution.
* JVM unsafe exceptions:
 * StackOverflowError
 * VirtualMachineError
 * ThreadDeath
 * LinkageError

A good practice as seen in various sections is to set time limits explicitely, so timeout() + retry() will be your best mates especially to protect against network partitioning. The more data flows in the Stream the better it should be able to auto heal to keep a good service availability.

> In Reactive Streams, at most one error can traverse a pipeline, so you can’t really double onError(e) a Subsriber, in theory. In practice we implemented the Rx operators retry() and retryWhen() that will cancel/re-subscribe onError. That means we still respect the contract as an entire new pipeline will be materialized transparently, with fresh action instances. That also means stateful Action like buffer() should be used with caution in this scenario since we just de-reference them, their state might be lost. We are working on alternatives, one of them involving external persistence for safe stateful Actions. A glimpse of that can be read in the related section.

**Fallback cascade fun**

```
Broadcaster<String> broadcaster = Broadcaster.create();

Promise<List<String>> promise =
    broadcaster
        .timeout(1, TimeUnit.SECONDS, Streams.fail(new Exception("another one!")))     （1）
        .onErrorResumeNext(Streams.just("Alternative Message"))      （2）
        .toList();

broadcaster.onNext("test1");
broadcaster.onNext("test2");
Thread.sleep(1500);

try {
  broadcaster.onNext("test3");
} catch (CancelException ce) {
  //Broadcaster has no subscriber, timeout disconnected the pipeline
}

promise.await();

assertEquals(promise.get().get(0), "test1");
assertEquals(promise.get().get(1), "test2");
assertEquals(promise.get().get(2), "Alternative Message");
```

1. TimeoutAction can fallback when no data is emited for the given time period, but in this case it will just emit another Exception…
1. …However, we are lucky to have onErrorResumeNext(Publisher) to catch this exception and actually deliver some String payload

Another classic example of fault-tolerant pipeline can be found in Recipes Section.

**Table 18. Handling errors**

|	Stream<T> API	|
|-------------------|
|	Role	|
|		|
|	when(Class<Throwable>, Consumer<Throwable>)	|
|	Observe specific exception types (and their hierarchy) coming from onError(Throwable).	|
|	oberveError(Class<Throwable>, BiConsumer<Object,Throwable>)	|
|	Similar to when but allows introspection of the failing onNext(Object) if any when the exception originally rose.	|
|	onErrorReturn(Class<Throwable, Function<Throwable,T>)	|
|	Provide a fallback signal T given an exception matching the passed type otherwise any exception. Commonly use in self-healing services.	|
|	onErrorResume(Class<Throwable, Publisher<T>)	|
|	Provide a fallback sequence of signal T given an exception matching the passed type otherwise any exception. Commonly use in self-healing services.	|
|	materialize() dematerialize()	|
|	Transform upstream signal into Signal<T>, and treat them as onNext(Signal<T>) signals. The immediate effect: it swallows error and completion signals, so it’s an effective way to process errors. Once errors are processed we can still run them by transforming theSignal<T> into the Reactive Streams right callback via dematerialize().	|
|	retry(int, Predicate<Throwable)	|
|	Cancel/Re-Subscribe the parent Stream up to the optional tries argument and matching the passed Predicate if provided.	|
|	retryWhen(Function<Stream<Throwable>,Publisher<?>>)	|
|	Cancel/Re-Subscribe the parent Stream when the returned Publisher from the passed Function emits onNext(Object). The function is called once on subscribe and the generated Publisher is subscribed. If the Publisher emits onError(e) oronComplete(), they will be propagated downstream. The Function receives a single Stream of errors which have occured in any subscribed pipeline. Can be combined with counting and delaying actions to provide for bounded and exponantial retry strategies.	|
|	recover(Class<Throwable>, Subscriber<Object>)	|
|	A retryWhen() shortcut to re-subscribe parent Publisher if the onError(Throwable) matches the given type. On recovery success, the passed Subscriber argument will receive the onNext(Object) that was the root signal associated with the exception, if any.	|
|	ignoreError(Predicate<Throwable>)	|
|	Transform the matching onError(Throwable) signals into onComplete(). If no argument has been provided, just transform any error into completion.	|
|	throw CancelException	|
|	That might be the only time we will mention anything related to exception bubbling up. However throwingCancelException.INSTANCE in any onNext(T) callback is a simple way to no-ack an incoming value and inform colocated (within the same thread stack) Publishers like Core Processor they might have to re-schedule this data later.	|