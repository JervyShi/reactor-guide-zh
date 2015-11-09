
# 微批次

> 把你未使用的CPU和内存更好的用来交换滥用导致的延时。

> — 克林贡谚语

翻阅过一两次101Stream的崩溃说明，勇敢的黑客们准备好一些快速的ROI（投资回报率）。实际上有效的调度与用于检测在每秒处理百万级消息待办事项的唯一标准相距甚远。

分布式系统的一个常见问题是由独立或有缓冲的IO写操作导致的延迟。当这种情况发生时，微批次或小块处理是将独立数据操作归类的方式。术语Micro（微量）的背后隐藏着一个更为具象化的名词：In Memory（内存中）。由于当前系统的速度仍然受限在光速内，主内存的读取依然比磁盘耗费时间更少。

```
延迟消耗比较数据

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

Streams是数据序列，因此找到可以切分缓冲区总量的范围是一个非常好的API。

**主要有两类定界：**

* Buffer（缓冲区）：依据范围将onNext(T)累加为一组List<T>传入子订阅者。
 * 最好使用需要把Iterable<T>作为入参的外部API。
* Window（窗口）：分离边界转发`onNext(T)`为不同`Stream<T>`并传递给子订阅者。
 * 最好使用如`Reduce`或任何订阅者/行为的积聚者反应到`onComlete()`。
 * 可以结合`flatMap`或`concatMap`在常规`Stream<T>`的独立窗口中做合并。

