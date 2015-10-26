# Request/Reply

Using an **EventBus** to publish and respond to events using Request/Reply.

It’s often the case that you want to receive a reply from a task executed on an EventBus’s configured **Dispatcher**. Reactor’s EventBus provides for more event handling models beyond the simple publish/subscribe model. You can also register a Function rather than a Consumer and have the EventBus automatically notify a replyTo key of the return value of the Function. Rather than using the `.on()` and `.notify()` methods, you use the `.receive()`, and `.send()` methods.

**Request/Reply**

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

1. Assign a Consumer to handle all replies indiscriminantly.
1. Assign a Function to perform work in the Dispatcher thread and return a result.
1. Publish an Event into the bus using the given replyTo key.

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