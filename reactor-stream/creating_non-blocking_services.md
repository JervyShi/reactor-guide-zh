
### Creating Non-Blocking Services

The first step is to isolate the microservice access. Instead of returning a type T or Future<T>, we will now start using Publisher<T> and specifically Stream<T> or Promise<T>. The immediate benefit is we don’t need to worry anymore about error handling and threading (yet): Errors are propagated in onError calls (no bubble up), threading might be tuned later for instance using dispatchOn. The additional bonus is we get to make our code more functional. It also works nice with Java 8 Lambdas! The target will be to reduce control brackets noise (if, for, while…) and limit more the need for sharing context. Ultimately our target design will encourage streaming over polling large datasets : functions will apply to a sequence, result by result, avoiding loop duplication.

> We prefer to use the implementation artefacts and not Publisher<T> to get compile-time access to all the Reactor Stream API, unless we want to be API agnostic (a possible case for library developers). Streams.wrap(Publisher<T>) will do the trick anyway to convert such generic return type into a proper Stream<T>.

**Table 15. Evolving to reactive microservices, part 1, error isolation and non-blocking in some UserService
**

|The Not So Much Win|The Win|
|----|-----|
|``` code 1 ```|``` code 2 ```|

code 1:
```
public User get(String name)
throws Exception {
  Result r = userDB.findByName(name);
  return convert(r);
}

public List<User> allFriends(User user)
throws Exception {
  ResultSet rs = userDB.findAllFriends(user);
  return convertToList(r);
}

public Future<List<User>> filteredFind(String name)
throws Exception {
  User user = get(name);
  if(user == null || !user.isAdmin()){
    return CompletedFuture.completedFuture(null);
  } else {
    //could be in an async thread if wanted
    return CompletedFuture.completedFuture(allFriends(user));
  }
}
```

code 2:
```
public Promise<User> get(final String name) {
  return Promises
    .task( () -> userDB.findByName(name))
    .timeout(3, TimeUnit.Seconds)
    .map(this::convert)
    .subscribeOn(workDispatcher());
}

public Stream<User> allFriends(final User user)  {
  return Streams
    .defer(() ->
      Streams.just(userDB.findAllFriends(user)))
    .timeout(3, TimeUnit.Seconds)
    .map(this::convertToList)
    .flatMap(Streams::from)
    .dispatchOn(cachedDispatcher());
    .subscribeOn(workDispatcher());
}

public Stream<User> filteredFind(String name){
    return get(name)
      .stream()
      .filter(User::isAdmin)
      .flatMap(this::allFriends);
}
```

**The Result**
* In all query methods:
 * No more throws Exception, its all passed in the pipeline
 * No more control logic, we use predefined operators such as map or filter
 * Only return Publisher (Stream or Promise)
 * Limit blocking queries in time with timeout (can be used later for retrying, fallback etc)
 * Use a pooled workDispatcher thread On Subscribe
* In get(name):
 * Use of typed single data Publisher, or Promise.
 * On Subscribe, call the task callback
* In allFriends(user):
 * Use defer() to invoke the DB query on the onSubscribe thread, lazily
 * No backpressure strategy yet and we read all the results in one blocking (but async) call
 * We convert returned list into a data stream in FlatMap
 * Dispatch each signal on an async dispatcher so downstream processing doesn’t negatively impact the read
* In filteredFind(name):
 * We convert a Promise from first get to a Stream with stream()
 * We only call allFriends() sub-stream if there is a valid user
 * The returned Stream<User> resume on the first allFriend() signal

