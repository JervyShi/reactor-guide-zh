# Architecture Overview

![Figure 1. The main modules present in Reactor 2.0](http://projectreactor.io/docs/reference/images/modules.png)
**Figure 1. The main modules present in Reactor 2.0**

The Reactor codebase is divided into several submodules to help you pick the ones that suit your needs while not burdening you with functionality you don’t need.

Following are some examples of how one might mix-and-match reactive technologies and Reactor modules to achieve your asynchronous goals:

* Spring XD + Reactor-Net (Core/Stream) : Use Reactor as a Sink/Source IO driver.
* Grails | Spring + Reactor-Stream (Core) : Use Stream and Promise for background Processing.
* Spring Data + Reactor-Bus (Core) : Emit Database Events (Save/Delete/…).
* Spring Integration Java DSL + Reactor Stream (Core) : Microbatch MessageChannel from Spring Integration.
* RxJavaReactiveStreams + RxJava + Reactor-Core : Combine rich composition with efficient asynchronous IO Processor
* RxJavaReactiveStreams + RxJava + Reactor-Net (Core/Stream) : Compose input data with RxJava and gate with Async IO drivers.

![Figure 2. A quick overview of how Reactor modules depend on one another](http://projectreactor.io/docs/reference/images/overview.png)
**Figure 2. A quick overview of how Reactor modules depend on one another**


