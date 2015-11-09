# 取消任务

有时你想取消一个任务，使它停止对事件通知的响应。如`.on()`和`.receive()`的注册方法会返回一个注册对象，如果此对象的引用存在，就可以使用它来取消给定选择器（selector）的消费者或函数。

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

1. Publish an event to the given topic. Should print `Event.toString()` in the console.
1. Cancel the Registration to prevent further events from reaching the Consumer.
1. Nothing should happen as a result of this notification.

> Keep in mind that cancelling a Registration involves accessing the internal Registry in an atomic way. In a system in which a large number of events are flowing into Consumers, it’s likely that your Consumer or Function might see some values after you’ve invoked the `.cancel()` method, but before the Registry has had a chance to clear the caches and remove the Registration. The `.cancel()` method could be described as a "request to cancel as soon as possible".

> You’ll notice this behavior right away in test classes where there’s no time delay between the `.on()`, `.notify()`, and `.cancel()` invocations.
