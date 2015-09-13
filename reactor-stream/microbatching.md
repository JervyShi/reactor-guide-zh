
# MicroBatching

> Better trade your unused CPU and Memory for your overused Latency

> — Klingon Proverb

After one or two reads of the 101 Stream crash intro, you courageous hacker are ready for some quick ROI. In effect dispatching efficiently is far away from the only item to check in the way of millions messages per sec todo list.

A common issue in Distributed Systems lies in the latency cost over indivudual vs buffered IO writes. When such situation arises, MicroBatching or small chunk-processing is the action to group individual data operations. Behind the term Micro hides a more concrete behavior named In Memory. Since the Speed of Light is still a limitation of systems today, main memory remains cheaper to read than disk.

```
Latency Comparison Numbers

L1 cache reference                            0.5 ns
Branch mispredict                             5   ns
L2 cache reference                            7   ns             14x L1 cache
Mutex lock/unlock                            25   ns
Main memory reference                       100   ns             20x L2 cache, 200x L1 cache
Compress 1K bytes with Zippy              3,000   ns
Send 1K bytes over 1 Gbps network        10,000   ns    0.01 ms
Read 4K randomly from SSD*              150,000   ns    0.15 ms
Read 1 MB sequentially from memory      250,000   ns    0.25 ms
Round trip within same datacenter       500,000   ns    0.5  ms
Read 1 MB sequentially from SSD*      1,000,000   ns    1    ms  4X memory
Disk seek                            10,000,000   ns   10    ms  20x datacenter roundtrip
Read 1 MB sequentially from disk     20,000,000   ns   20    ms  80x memory, 20X SSD
Send packet CA->Netherlands->CA     150,000,000   ns  150    ms

Notes
-----
1 ns = 10-9 seconds
1 ms = 10-3 seconds
* Assuming ~1GB/sec SSD

Credit
------
By Jeff Dean:               http://research.google.com/people/jeff/
Originally by Peter Norvig: http://norvig.com/21-days.html#answers
```

Streams are sequences of data, so finding boundaries to cut aggregated buffers is an out-of-the box API.

**There are two categories for delimitations:**

* Buffer : Concrete boundaries accumulating onNext(T) inside grouped List<T> passed to the child Subscriber.
 * Used best with external API requiring Iterable<T> input argument.
* Window : Discrete boundaries forwarding onNext(T) into distinct Stream<T> passed to the child Subscriber.
 * Used best with accumulators such as reduce or any subscriber/action reacting to onComplete().
 * Can be combined with flatMap or concatMap which merge back the individual windows in a common Stream<T>

