
### Composing multiple Services Calls

In this second step, we will expand our thinking to the consuming aspect. In a transition phase, keep in mind that Stream can be blocked using operators.

There are two issues to address in target: robustness (network partition tolerance etc) and avoiding to wait for a service before processing another:

**Table 16. Evolving to reactive microservices, part 2, parallel requests and resiliency in some RickAndMortyService**

|The Not So Much Win|The Win|
|----|----|
|``` code 1 ```|``` code 2 ```|

code 1:
```
int tries = 0;
while(tries < 3){
  try{
    Future<List<User>> rickFriends =
      userService.fitleredFind("Rick");

    Future<List<User>> mortyFriends =
      userService.fitleredFind("Morty");

    System.out.println(
      rickFriends.get(3, TimeUnit.SECONDS)
      .addAll(
        mortyFriends.get(3, TimeUnit.SECONDS))
    );

  }catch(Exception e){
    if(tries++ >= 3) throw e;
    Thread.sleep(tries*1000);
  }
}
```

code 2:
```
return Streams.merge(
  userService.filteredFind("Rick"),
  userService.filteredFind("Morty")
)
.buffer()
.retryWhen( errors ->
  errors
  .zipWith(Streams.range(1,3), t -> t.getT2())
  .flatMap( tries -> Streams.timer(tries) )
)
.consume(System.out::println);
```

**The Result**

* Streams.merge() is a non-blocking coordinating operation mixing the two queries in one
* buffer() will aggregate all results until completion or error (which we timed previously)
* retryWhen(Function<Stream<Throwable>, Publisher<?>> will keep re-subscribing if an error is propagated
 * zipWith will combine errors with a number of tries up to 3 times
 * zipWith only return the number of tries from the tuple
 * flatMap + Streams.timer(long) convert each try into a delayed signal (seconds by default)
 * Each time a signal is sent by this returned Publisher, cancel and subscribe again, until a onComplete or onError is sent.
 * flatMap only completes if the internal timer AND the upstream have completed, so after the range of 3 or after errors sequence itself terminates.
