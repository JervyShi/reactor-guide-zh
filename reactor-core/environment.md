# Environment
Environment由Reactor的使用者（或者扩展模块，例如：@Spring）来创建和终止的。它自动从`META_INF/reactor/reactor-environment.properties`路径下读取配置文件。

> Properties文件可在运行时被修改，通过提供一个放置于classpath路径下`META-INF/reactor`中的新Properties文件。

运行时默认配置的切换是通过修改以下的环境变量来实现：reactor.profiles.active。
```
java - jar reactor-app.jar -Dreactor.profiles.active=turbo
```

启动和终止Environment

```
Environment env = Environment.initialize();

//Current registered environment is the same than the one initialized
Assert.isTrue(Environment.get() == env);

//Find a dispatcher named "shared"
Dispatcher d  = Environment.dispatcher("shared");

//get the Timer bound to this environment
Timer timer = Environment.timer();

//Shutdown registered Dispatchers and Timers that might run non-daemon threads
Environment.terminate();
//An option could be to register a shutdownHook to automatically invoke terminate.```

> 在一个JVM应用中最好只有一个**Environment**。大多数情况下可以使用`Environment.initializeIfEmpty()`。


