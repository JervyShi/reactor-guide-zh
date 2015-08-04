# Requirements

* Reactor运行需要Java 7及以上版本。

 * 但完整的功能组合表达式需要java8的lambdas支持。

 * 作为后备，支持spring、clojure和groovy的扩展。

* Reactor需要JVM支持Unsafe获取方式时才能发挥最强的能力(例如, ：Android不支持).

 * 如果不支持Unsafe访问时所有基于RingBuffer的特性将会消失。

* Reactor被打包成JAR并存放于Maven中央仓库，你可以使用喜欢的构建工具来拉取这个依赖包。
