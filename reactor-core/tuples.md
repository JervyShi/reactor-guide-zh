# Tuples

You might have noticed these interfaces are strongly typed with Generic support and a small fixed number of argument. So how do you pass more than 1 or 2 arguments ? The answer is in one class : Tuple. Tuples are like typed CSV lines in a single object instance, you want them in functional programming to keep both the type safety and a variable number of arguments.

Let’s take the previous example and try replacing the double-argument BiConsumer with a single-argument Consumer:

```
Consumer<Tuple2<Consumer<String>, String>> biConsumer = tuple -> {
        for(int i = 0; i < 10; i++){
                //Correct typing, compiler happy
                tuple.getT1().accept(tuple.getT2());
        }
};

biConsumer.accept(
        Tuple.of(
                consumer,
                transformation.apply(supplier.get())
        )
);
```

> *Tuples involve a bit more allocation, and that’s why the common use cases of comparison or keyed signals are handled with Bi** artifacts directly.*


