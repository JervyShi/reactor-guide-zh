# reactor-core

> You should never do your asynchronous work alone.

> — Jon Brisbin

> After writing Reactor 1

> You should never do your asynchronous work alone.

> — Stephane Maldini

> After writing Reactor 2

Head first with a Groovy example of some Core work

```
//Initialize context and get default dispatcher
Environment.initialize()

//RingBufferDispatcher with 8192 slots by default
def dispatcher = Environment.sharedDispatcher()

//Create a callback
Consumer<Integer> c = { data ->
        println "some data arrived: $data"
}

//Create an error callback
Consumer<Throwable> errorHandler = { it.printStackTrace }

//Dispatch data asynchronously
dispatcher.dispatch(1234, c, errorHandler)

Environment.terminate()
```

A second taster, the Reactive Streams way

```
//standalone async processor
def processor = RingBufferProcessor.<Integer>create()

//send data, will be kept safe until a subscriber attaches to the processor
processor.onNext(1234)
processor.onNext(5678)

//consume integer data
processor.subscribe(new Subscriber<Integer>(){

  void onSubscribe(Subscription s){
    //unbounded subscriber
    s.request Long.MAX
  }

  void onNext(Integer data){
    println data
  }

  void onError(Throwable err){
    err.printStackTrace()
  }

  void onComplete(){
    println 'done!'
  }
}

//Shutdown internal thread and call complete
processor.onComplete()
```

