# 发布/订阅

使用`EventBus`来发布，使用发布/订阅来响应事件。

Reactor的`EventBus`允许你注册一个消费者用于当满足某些条件的Key收到通知时来处理事件。这个功能是通过`Selector`来做的。它类似于订阅主题，通过`Reactor`中`Selector`的实现来匹配多种判断依据，例如从`Class<?>`类型到正则，到Json路径表达式。这是一种十分灵活和强大的抽象，这种抽象提供了广泛的可能性。

同一个`Selector`可以注册在多个`Consumer`上，多个`Selector`可以匹配同一个key。这种方式很容易做聚集和广播：你可以简单的在同一个主题`Selector`上订阅多个消费者。

> 如果你更新到`Reactor`1.1版本，你会发现`Reactor`类已经不存在了。它被更名为`EventBus`，来更准确反馈它在框架中的角色。

**使用`Selector`来处理事件**

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