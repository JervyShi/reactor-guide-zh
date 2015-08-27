# Timers
Dispatcher尽可能快的计算接收到的任务。Timers提供周期性或者一次性的调度API。Reactor Core提供的默认定时器是`HashWheelTimer`，它会自动绑定到任何新的Environment中。`HashWheelTimer`对处理大量的、并发的、内存调度任务有巨大的优势，它是java TaskScheduler的一个强大的替代品。

> HashWheelTimer是适合一些小周期任务，这不是一个持久化的调度器，应用关闭时会丢失所有任务。

> 下个Release版本Timer将会有一些值得关注的更新，例如：我们想要增加基于Redis的可持久化/共享的调度支持。请在这里给出您的任何意见或建议！

在我们的Groovy Spock测试中创建一个简单的Timer示例：

```
import reactor.fn.timer.Timer

//...

given: "a new timer"
    Environment.initializeIfEmpty()
    Timer timer = Environment.timer()
    def latch = new CountDownLatch(10)

when: "a task is submitted"
    timer.schedule(
            { Long now -> latch.countDown() } as Consumer<Long>,
            period,
            TimeUnit.MILLISECONDS
    )

then: "the latch was counted down"
    latch.await(1, TimeUnit.SECONDS)
    timer.cancel()
    Environment.terminate()
```

