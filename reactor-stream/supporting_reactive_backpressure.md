
### Supporting Reactive Backpressure

In this last step, we pay a visit to the UserService.allFriends query which is right now polling entire datasets from Database.

**Table 17. Evolving to reactive microservices, part 3, backpressure in UserService.allFriends**

|The Win|The Epic Win|
|----|----|
|``` code 1 ```|``` code 2 ```|

code 1:
```
return Streams
  .defer(() ->
    Streams.just(userDB.findAllFriends(user)))
  .timeout(3, TimeUnit.Seconds)
  .map(this::convertToList)
  .flatMap(Streams::from)
  .dispatchOn(cachedDispatcher());
  .subscribeOn(workDispatcher());

stream
  .buffer()
  .consume(System.out::println);
```

code 2:
```
return Streams
  .createWith(
    (demand, sub) -> {
      ResultSet rs = sub.context();
      long cursor = 0l;

      while(rs.hasNext()
        && cursor++ < demand
        && !sub.isCancelled()){

        sub.onNext(rs.next());
      }

      if(!rs.hasNext()){
        sub.onComplete();
      }
    },
    sub -> userDB.findAllFriends(user),
    resultSet -> resultSet.close()
  )
  .timeout(3, TimeUnit.Seconds)
  .map(this::convert)
  .dispatchOn(cachedDispatcher());
  .subscribeOn(workDispatcher());

stream
  .buffer(5, 200, TimeUnit.MILLISECONDS)
  .consume(System.out::println);
```

**The Result**

* Yes it’s more verbose…
* …But now we stream result by result from the query (could have used pagination with SQL limits as well).
* Streams.createWith is a PublisherFactory which intercepts requests, start and stop.
 * The request consumer gives precisely how many elements a subscriber is ready to receive.
 * The request consumer receives a SubscriberWithContext delegating to the real Subscriber, it gives access to shared context and cancel status.
 * We send at most as many individual Result as demanded
 * We complete when the query read is fully processed
* Since the data is individual now, convertToList is unecessary, replaced with convert
* The Consuming aspect can start using tools such as capacity(long) or buffer(int) to batch consume the request 5 by 5.
 * As a result the flow will be perceived faster because we don’t print after every rows have been read
 * We add a time limit to the batch since it might not match with the size

> It’s important to balance the use of stateful Iterable<T> like List<T> vs individual streaming T. A List might incur at some point more latency since we take more time to create it. It’s also not playing that well in favor of resiliency since it’s a whole batch we can lose if a fatal error occurs. Finally, streaming T data makes sizing demand more predictable because we can score individual signals instead of batches of signals.


