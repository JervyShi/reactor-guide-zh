# Organizing Functional Blocks

Every Functional component gives the explicit intent of its general mission:

* Consumer: simple callback - fire-and-forget
* BiConsumer: simple callback with two arguments (often used in sequence comparaisons, e.g. previous and next arguments)
* Function: transforming logic - request/reply
* BiFunction: transforming with two arguments (often used in accumulators, comparing previous and next arguments then returning a new value)
* Supplier: factory logic - polling
* Predicate: testing logic - filtering

> *We consider Publisher and Subscriber interfaces also functional blocks, dare we say Reactive Functional Blocks. Nevertheless they are the basic components used everywhere around in Reactor and Beyond. Stream API will usually accept reactor.fn arguments to create on your behalf the appropriate Subscribers.*

**The good news about wrapping executable instructions within Functional artefacts is that you can reuse them like Lego Bricks.**

```
Consumer<String> consumer = new Consumer<String>(){
        @Override
        void accept(String value){
                System.out.println(value);
        }
};

//Now in Java 8 style for brievety
Function<Integer, String> transformation = integer -> ""+integer;

Supplier<Integer> supplier = () -> 123;

BiConsumer<Consumer<String>, String> biConsumer = (callback, value) -> {
        for(int i = 0; i < 10; i++){
                //lazy evaluate the final logic to run
                callback.accept(value);
        }
};

//note how the execution flows from supplier to biconsumer
biConsumer.accept(
        consumer,
        transformation.apply(
                supplier.get()
        )
);
```

It might not sound like a striking revolution at first, however this basic mindset change will reveal precious for our mission to make asynchronous code sane and composable. The Dispatchers will use Consumer for their typed Data and Error callbacks. The Reactor Streams module will use all these artifacts for greater good as well.

> *A good practice when using an IoC container such as Spring is to leverage the Java Configuration feature to return stateless Functional Beans. Then injecting the blocks in a Stream pipeline or dispatching their execution becomes quite elegant.*


