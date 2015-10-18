
## 理解线程模型

`Reactive Streams`和`Reactive`扩展的一个共同目标是通过信号回调这种方式不再武断的遵循线程习惯。它会在现在和某个时刻`T`执行是`Streams`的所有。非同步信号也可以保存`Subscriber`的并发访问（无共享），但是信号和请求可以在两个不对称的线程上执行。

默认情况下，`Stream`被分配了一个`SynchronousDispatcher`，并且将会通过`Stream.getDispatcher()`来通知它直接子级。

> 多种多样的`Stream`工厂，`Broadcaster`，`Stream.dispatchOn`和终端的`xxxOn`方法可能会修改默认的`SynchronousDispatcher`

**Reactor Stream中三大主要可用线程切换的基本了解：**

* `Stream.dispatchOn`的作用是Stream下唯一可用的方法用于在给定的dispatcher上分发`onError`，`onComplete`和`onNext`信号。
 * Processor的行为不支持并发分发，例如`WorkQueueDispatcher`。
 * request和cancel将会在dispatcher上执行，如果它的上下文准备完毕。否则它将会在当前dispatch执行完毕后执行。
* `Stream.subscribeOn`将只会在已经通过的dispatcher上执行。
 * 由于唯一一次通过的Dispatcher被称之为`onSubscribe`，任何dispatcher可以使用并发分发器，类似`WorkQueueDispatcher`。
 * 第一次请求可能仍然在`onSubscribe`线程中执行，如`Stream.consume()`操作的实例。
* `Stream.process`附属的`Processor`实例也会影响线程。类似`RingBufferProcessor`的`Processor`将会使用它自己管理的线程来执行`Subscriber`。
 * 如果上下文准备完毕，请求和取消将会在同一个processor上执行。
 * `RingBufferWorkProcessor`仅最多分发`onNext`信号到一个`Subscriber`上，除非它在执行过程中被中断（重播给一个新的`Subscriber`）。

常规订阅是由`onSubscribe`开始请求数据，`subscribeOn`是一种有效的工具来放大`Stream`，尤其是无界的。如果一个订阅者向`onSubscribe`请求`Long.MAX_VALUE`条数据，它将会成为唯一一个执行请求者，并且它会在分配给这个订阅者的`diapatcher`上运行。这是无界`Stream`的默认消费行为。

**在无限请求的线程中跳转**

```
Streams
  .range(1, 100)
  .dispatchOn(Environment.sharedDispatcher())   (2)
  .subscribeOn(Environment.workDispatcher())    (1)
  .consume();   (3)
```

1. 分配一个`onSubscribe`工作队列`dispatcher`。
1. 分配`onNext`，`onError`，`onComplete`信号的`dispatcher`。
1. 使用`Subscription.request(Long.MAX)`来消费`Stream` `onSubscribe`

![Figure 12. 无界消费者的subscribeOn和dispatchOn/process](http://projectreactor.io/docs/reference/images/longMaxThreading.png)
**Figure 12. 无界消费者的subscribeOn和dispatchOn/process**

然而，当多于1个请求将会变的复杂时，`subscribeOn`会变的比较无用，例如在限定`Stream.capacity(n)`的分步消费中。唯一执行请求的线程可能运行在分配给`subscribeOn`的第一个`dispatcher`上。

**在有限请求的线程中跳转**

```
Streams
  .range(1, 100)
  .process(RingBufferProcessor.create())    (2)
  .subscribeOn(Environment.workDispatcher())    (1)
  .capacity(1);     (3)
  .consume();   (4)
```

1. Assign an onSubscribe work queue dispatcher. Note that it is placed after process as the subscribeOn will run on the ringBuffer thread on subscriber and we want to alter it to the work dispatcher.
1. Assign an async signal onNext, onError, onComplete processor. Similar to dispatchOn behavior.
1. Assign a Stream capacity to 1 so the downstream action adapts
1. Consume the Stream onSubscribe with Subscription.request(1) and after every 1 onNext.

![Figure 13. subscribeOn and dispatchOn/process with an bounded (demand N < Long.MAX) Subscriber](http://projectreactor.io/docs/reference/images/nThreading.png)
**Figure 13. subscribeOn and dispatchOn/process with an bounded (demand N < Long.MAX) Subscriber**

