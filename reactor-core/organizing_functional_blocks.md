# 组织功能模块

每个功能模块为它常规任务提供的明确意图：

* Consumer: 简单回调 - 一劳永逸
* BiConsumer: 包含两个入参的简单回调（通常用于序列比较，例如：前一个和后一个参数）
* Function: 转换逻辑 - 输入/输出
* BiFunction: 两个参数的转换逻辑（通常用在累加器，比较前一个和下一个参数，返回一个新的值。）
* Supplier: 工厂逻辑 - 轮询
* Predicate: 测试逻辑 - 过滤

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


