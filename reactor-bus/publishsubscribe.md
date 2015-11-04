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
1. 通过静态的`Environment`使用默认，共享的`RingBufferDispatcher`来创建`EventBus`。
2. 分配一个消费者用于在`EventBus`被通知指定key匹配到这个`Selector`时调用。
3. 使用给定的主题向`EventBus`发布一个事件。

> 简写静态方法`$`只是等同于`Selectors.object()`的一个便利的助手。有些人不喜欢使用简写的方法，例如：`$()`为`ObjectSelector`，`R()`为`ObjectSelector`，`T()`为`ObjectSelector`等等。`Selector`类有与简写对应的长方法名，简写是为了减少无用代码的干扰，并使代码更具可读性。