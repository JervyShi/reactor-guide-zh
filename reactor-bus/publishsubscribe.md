# 发布/订阅

使用`EventBus`来发布，使用发布/订阅来响应事件。
Using an EventBus to publish and respond to events using Publish/Subscribe.

Reactor’s EventBus allows you to register a Consumer to handle events when the notification key matches a certain condition. This assignment is achieved via the Selector. It’s similar to subscribing to a topic, though Reactor’s Selector implementations can match on a variety of critera, from Class<?> type to regexes, to JsonPath expressions. It is a very flexible and powerful abstraction that provides a wide range of possibilities.

You can register multiple Consumers using the same Selector and multiple Selectors can match a given key. This way it’s easy to do aggregation and broadcasting: you simply subscribe multiple Consumers to the same topic Selector.

> If you’re upgrading from Reactor 1.1, you’ll see that the Reactor class no longer exists. It has been renamed to EventBus to more accurately reflect its role in the framework.

**Handling events using a Selector**

```
EventBus bus = EventBus.create(Environment.get());  (1) 

bus.on($("topic"), (Event<String> ev) -> {
  String s = ev.getData();
  System.out.printf("Got %s on thread %s%n", s, Thread.currentThread());
});     (2)

bus.notify("topic", Event.wrap("Hello World!"));    (3)
```

1. Create an EventBus using the default, shared RingBufferDispatcher from the static Environment.
2. Assign a Consumer to invoke when the EventBus is notified with a key that matches the Selector.
3. Publish an Event into the EventBus using the given topic.

> The shorthand static method $ is just a convenience helper that is identical to Selectors.object(). Some people don’t like to use the shorthand methods like $() for ObjectSelector, R() for RegexSelector, T() for ClassSelector, and so forth. The Selectors class has longer method name alternatives for these shorthand versions, which are simply aliases to reduce code noise and make reactive code a little more readable.