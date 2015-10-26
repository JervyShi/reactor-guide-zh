# Cancelling a Task

Sometimes you want to cancel a task to cause it to stop responding to event notifications. The registration methods `.on()` and `.receive()` return a Registration object which, if a reference to it is held, can be used later to cancel a Consumer or Function for a given Selector.

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
