# Core Overview
![Figure 5. How Doge can use Reactor-Core](http://projectreactor.io/docs/reference/images/core-overview.png)
**Figure 5. How Doge can use Reactor-Core**

**Reactor Core** has the following artefacts:

* Common IO & functional types, some directly backported from Java 8 Functional Interfaces

 * Function, Supplier, Consumer, Predicate, BiConsumer, BiFunction

 * Tuples

 * Resource, Pausable, Timer

 * Buffer, Codec and a handful of predifined Codecs
* Environment context

* Dispatcher contract and a handful of predefined Dispatchers

* Predefined Reactive Streams Processor

Alone, reactor-core can already be used as a drop-in replacement for another Message-Passing strategy, to schedule timed tasks or to organize your code in small functional blocks implementing the Java 8 backport interfaces. Such breakdown allows to play more nicely with other Reactive libraries especially removing the burden of understanding the RingBuffer for the impatient developer.

*Reactor-Core implicitely shadows LMAX Disruptor, so it won’t appear nor collide with an existing Disruptor dependency*




