# reactor-core模块

> You should never do your asynchronous work alone.

> — Jon Brisbin

> 完成Reactor 1后写到

> You should never do your asynchronous work alone.

> — Stephane Maldini

> 完成Reactor 2后写到

首先，我们来用一个Groovy的示例来展示Core模块的功能：

```
//初始化上下文，获取默认dispather
Environment.initialize()

//使用默认8192 slots的RingBufferDispatcher
def dispatcher = Environment.sharedDispatcher()

//创建一个回调
Consumer<Integer> c = { data ->
        println "some data arrived: $data"
}

//创建一个异常回调
Consumer<Throwable> errorHandler = { it.printStackTrace }

//异步调度数据
dispatcher.dispatch(1234, c, errorHandler)

Environment.terminate()
```

然后，用Reactive Streams的方式来实现

```
//独立的异步processor
def processor = RingBufferProcessor.<Integer>create()

//发送数据，将会保持数据安全直到subscriber连接到processor为止
processor.onNext(1234)
processor.onNext(5678)

//消费整型数据
processor.subscribe(new Subscriber<Integer>(){

  void onSubscribe(Subscription s){
    //无界限的subscriber
    s.request Long.MAX
  }

  void onNext(Integer data){
    println data
  }

  void onError(Throwable err){
    err.printStackTrace()
  }

  void onComplete(){
    println 'done!'
  }
}

//关闭内部线程，调用complete函数
processor.onComplete()
```

