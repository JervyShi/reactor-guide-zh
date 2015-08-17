# 元组（Tuples）

你可能会注意到那些接口，它们对输入参数和比较少的固定数量的参数的泛型有很好的支持。那么如何传递超过1个或者两个的入参呢？答案是使用Tuple。Tuple类似与CSV中每行指定一个对象的实例，你需要它们在函数式编程中实现即保持类型安全又可以使用多个数量的参数。

让我们看下之前的例子，并且尝试用单个参数的Consumer来替代两个参数的BiConsumer：

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

> *Tuple需要分配更多的内存空间，因此在需要比较和键值对的场景下推荐直接使用Bi** 组件。*

