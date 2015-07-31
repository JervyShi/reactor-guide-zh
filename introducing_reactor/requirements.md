# Requirements

* Reactor needs at minimum Java 7 to execute.

 * But the full expressive potential of functional composition happens with Java 8 Lambdas.

 * As a fallback have a look at Spring, Clojure or Groovy extensions.

* Reactor runs at full capacity when the JVM supports Unsafe access (e.g., not the case for Android).

 * All RingBuffer based features will not work when Unsafe is missing.

* Reactor is packaged as traditional JAR archives in Maven Central and can be pulled into any JVM project as a dependency using your preferred build tool.
