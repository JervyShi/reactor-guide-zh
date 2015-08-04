# Architecture Overview

![图 1. Reactor 2.0主要模块](http://projectreactor.io/docs/reference/images/modules.png)
**图 1. Reactor 2.0主要模块**

Reactor的代码库是将整体功能拆分成多个子模块，你可以按需选择适合你需求的模块而不必将你不需要的模块集成在系统中。

下面是一些混合组合reactive技术和使用Reactor模块实现异步的目标的例子：

* Spring XD + Reactor-Net (Core/Stream) : 使用Reactor作为Sink/Source IO驱动。
* Grails | Spring + Reactor-Stream (Core) : 使用Stream和Promise作为后台处理程序。
* Spring Data + Reactor-Bus (Core) : 数据库Emit事件 (Save/Delete/…)。
* Spring Integration Java DSL + Reactor Stream (Core) : 基于Spring Integration的Microbatch MessageChannel。
* RxJavaReactiveStreams + RxJava + Reactor-Core : Combine rich composition with efficient asynchronous IO Processor
* RxJavaReactiveStreams + RxJava + Reactor-Net (Core/Stream) : Compose input data with RxJava and gate with Async IO drivers.

![图 2. Reactor模块依赖预览图](http://projectreactor.io/docs/reference/images/overview.png)
**图 2. Reactor模块依赖预览图**


