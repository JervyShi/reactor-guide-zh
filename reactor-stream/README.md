# reactor-stream

> Nope, you shall not use Future.get(), ever.

> — Stephane Maldini

> with a Banking Sector Customer

**Head first with a Java 8 example of some Stream work**

```
import static reactor.Environment.*;
import reactor.rx.Streams;
import reactor.rx.BiStreams;

//...

Environment.initialize()

//find the top 10 words used in a list of Strings
Streams.from(aListOfString)
  .dispatchOn(sharedDispatcher())
  .flatMap(sentence ->
    Streams
      .from(sentence.split(" "))
      .dispatchOn(cachedDispatcher())
      .filter(word -> !word.trim().isEmpty())
      .observe(word -> doSomething(word))
  )
  .map(word -> Tuple.of(word, 1))
  .window(1, TimeUnit.SECONDS)
  .flatMap(words ->
    BiStreams.reduceByKey(words, (prev, next) -> prev + next)
      .sort((wordWithCountA, wordWithCountB) -> -wordWithCountA.t2.compareTo(wordWithCountB.t2))
      .take(10)
      .finallyDo(event -> LOG.info("---- window complete! ----"))
  )
  .consume(
    wordWithCount -> LOG.info(wordWithCount.t1 + ": " + wordWithCount.t2),
    error -> LOG.error("", error)
  );
  ```