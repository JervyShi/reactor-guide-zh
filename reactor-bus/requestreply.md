# 请求/应答

使用`EventBus`来发布和响应请求/应答（Request/Reply）类事件。

常见的例子：你想要从在`EventBus`配置的分发器上执行的一个任务中收到应答。除了简单的发布/订阅模型外，`Reactor`的`EventBus`还提供多种事件处理模型。除了注册消费者之外，你也可以注册函数（`Function`），来让`EventBus`自动通知一个应答给函数返回值指定的键。在使用`.on()`和`.notify()`方法外，你也可以使用`.receive()`和`.send()`方法。

**请求/应答**

```
EventBus bus;

bus.on($("reply.sink"), ev -> {
  System.out.printf("Got %s on thread %s%n", ev, Thread.currentThread())
});     (1)

bus.receive($("job.sink"), ev -> {
  return doWork(ev);
});     (2)

bus.send("job.sink", Event.wrap("Hello World!", "reply.sink"));     (3)
```

1. 指定一个消费者来处理所有应答。
2. 指定一个方法在分发线程中执行任务并返回结果。
3. 使用给定的应答键在总线中发布一个事件。

If you don’t have a generic topic on which to publish replies, you can combine the request and reply operation into a single call using the `.sendAndReceive(Object, Event<?>, Consumer<Event<?>>)` method. This performs a `.send()` call and invokes the given replyTo callback on the Dispatcher thread when the Functions are invoked.

**sendAndReceive()**

```
EventBus bus;

bus.receive($("job.sink"), (Event<String> ev) -> {
  return ev.getData().toUpperCase();
});     (1) 

bus.sendAndReceive(
    "job.sink",
   Event.wrap("Hello World!"),
   s -> System.out.printf("Got %s on thread %s%n", s, Thread.currentThread())
);      (2)
```

1. Assign a Function to perform work in the Dispatcher thread and return a result.
1. Publish an Event into the bus and schedule the given replyTo Consumer on the Dispatcher, passing the receive Function’s result as input.