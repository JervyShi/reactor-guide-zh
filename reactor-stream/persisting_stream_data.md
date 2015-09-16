
## Persisting Stream Data

Not everything has to be in-memory and Reactor has started a story to integrate (optional dependency) with Java Chronicle.

```
return Streams.merge(
  userService.filteredFind("Rick"),
  userService.filteredFind("Morty")
)
.buffer()
.retryWhen( errors ->
  errors
  .zipWith(Streams.range(1,3), t -> t.getT2())
  .flatMap( tries -> Streams.timer(tries) )
)
.consume(System.out::println);
```

**Table 19. Persisting signals safely**

|	Functional API or Factory method	|
|----|
|	Role	|
|	Stream.onOverflowBuffer(CompletableQueue)	|
|		|
|	IOStreams.persistentMapReader()	|
|		|
|	IOStreams.persistentMap()	|