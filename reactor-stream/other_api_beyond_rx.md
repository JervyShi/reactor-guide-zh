
## Other API beyond Rx

In addition to implementing directly the Reactive Streams, some more Stream methods not covered differ or are simply not documented by Reactive Extensions.

**Table 22. Other methods uncovered in the previous use cases.**

|	```	Stream<T> API	```	|		Input Type		|		Output Type		|
|----|----|----|
|		Role		|				|				|
|				|				|				|
|	```	after()	```	|	```	T	```	|	```	Void	```	|
|		Only consume onComplete() and onError(Throwable) signals.		|				|				|
|	```	log(String)	```	|	```	T	```	|	```	T	```	|
|		Use SLF4J and the given category to log each signal.		|				|				|
|	```	split	```	|	```	Iterable<T>	```	|	```	T	```	|
|		Blocking transformation from Iterable<T> to as many onNext(T) as available.		|				|				|
|	```	sort(int, Comparator<T>)	```	|	```	T	```	|	```	T	```	|
|		Accept up to the given size into an in memory PriorityQueue, apply the Comparator<T> to sort, and emit all its pending onNext(T)signals.		|				|				|
|	```	combine()	```	|	```	I	```	|	```	O	```	|
|		Scan for the most ancient parent or Action, from right to left. As a result, it will create a new Processor with the input onXXXX signals dispatched to the old action and the output subscribe delegated to the current action. Example: code 1		|				|				|
|	```	keepAlive()	```	|	```	T	```	|	```	T	```	|
|		Prevent any Subscription.cancel() to propagate from the Subscriber.		|				|				|

**code 1:**

```
Action<Integer, String> processor = stream
  .filter( i -> i<2 )
  .map(Object::toString)
  .combine();

  processor.consume(System.out::println);
  processor.onNext(1);
  processor.onNext(3);
```