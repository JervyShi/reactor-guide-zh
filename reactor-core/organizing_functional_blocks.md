# 组织功能模块

每个功能模块为它常规任务提供的明确意图：

* Consumer: 简单回调 - 一劳永逸
* BiConsumer: 包含两个入参的简单回调（通常用于序列比较，例如：前一个和后一个参数）
* Function: 转换逻辑 - 输入/输出
* BiFunction: 两个参数的转换逻辑（通常用在累加器，比较前一个和下一个参数，返回一个新的值。）
* Supplier: 工厂逻辑 - 轮询
* Predicate: 测试逻辑 - 过滤

> *我们把Publisher和Subscriber的接口也作为函数式块，我们称之为Reactive函数式块。然而他们是使用Reactor和相关功能的基础组件。通常Stream API会接受*reactor.fn*参数来为你创建合适的Subscriber。

**好消息是函数式模块中可执行指令的包装就像乐高积木一样可以反复使用。**

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

首先听起来这并不像是一个引人瞩目的革命，但是这种基本思维模式的改变，将揭示我们使异步代码变的稳健和可组合性的使命是多么可贵。Diaspatcher将输入数据和错误回调交给Consumer来处理。Reactor Stream模块将更好的使用这些组件。

> *当使用类似Spring的IoC容器时，一个良好的实践是使用Java配置来返回一个无状态的函数式Bean。然后将这些块注入到Stream Pipeline或者分发他们的执行会变的十分优雅。*


