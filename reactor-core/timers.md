# Timers
Dispatchers compute incoming tasks as soon as possible. Timers however come with periodic and one-time scheduling API. Reactor Core offers an **HashWheelTimer** by default and it is automatically bound to any new Environment. HashWheelTimers are perfect for dealing with massive concurrent in-memory scheduled tasks, it’s a powerful alternative to Java **TaskScheduler**.

> While it is suited for windowing (mini tasks periods under the minute order of magnitude), it is not a resilient scheduler since all tasks are lost when the application shutdowns.

> Timers will receive some attention along the next releases, e.g. we would love to add persisting/shared scheduling support with Redis. Please voice your opinion or propose any contribution here!

A simple timer creation as seen in one of our Groovy Spock test:

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

