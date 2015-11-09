# 取消任务

有时你想取消一个任务，使它停止对事件通知的响应。如`.on()`和`.receive()`的注册方法会返回一个注册对象，如果此对象的引用存在，就可以使用它来取消给定选择器（`selector`）的消费者或函数。

```
EventBus bus;

Registration reg = bus.on($("topic"),
                          s -> System.out.printf("Got %s on thread %s%n", s, Thread.currentThread()));

bus.notify("topic", Event.wrap("Hello World!"));    (1)

// ...some time later...
reg.cancel();   (2)

// ...some time later...
bus.notify("topic", Event.wrap("Hello World!"));    (3)
```

1. 向指定主题发送一个事件，将会在控制台打印`Event.toString()`。
2. 取消注册，不再接收未来发送到消费者的事件。
3. 这次通知将会石沉大海。

> 记住取消注册包含对内部注册机制的原子性访问。在一个拥有大量事件流向消费者的系统中，消费者或者函数在你执行过`.cancel()`方法之后以及注册机还没有抓到机会清理掉缓存及清理掉注册之前，仍然可能看到一些事件。`.cancel()`方法可以被描述为“请求尽可能快的取消”。

> 
> You’ll notice this behavior right away in test classes where there’s no time delay between the `.on()`, `.notify()`, and `.cancel()` invocations.
