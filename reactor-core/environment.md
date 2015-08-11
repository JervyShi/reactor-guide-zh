# Environment
Environments are created and terminated by the reactor user (or by the extension library if available, e.g. @Spring). They automatically read a configuration file located in META_INF/reactor/reactor-environment.properties.

> Properties file can be tuned at runtime by providing under the classpath location META-INF/reactor a desired new properties configuration.

There switching from the default configuration at runtime is achieved by passing the followying Environment Variable: reactor.profiles.active.

```
java - jar reactor-app.jar -Dreactor.profiles.active=turbo
```

Starting and Terminating the Environment

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

> It’s best to try maintaining a single Environment alive for a given JVM application. Use of Environment.initializeIfEmpty() will be prefered most of the time.


