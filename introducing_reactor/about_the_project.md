# 项目简介

该项目起始于2012年，并经过较长时间的孵化。 Reactor 1.x 在2013年问世。此版本成功部署到各种组织，不仅有开源组织（如：Meltdown），还有商业机构（如：Pivotal RTI）。2014年我们合作开发实现新的“Reactive流标准”，并且在2015年4月开始版本2.0的大规模重构计划。Reactive流标准拉近了分发机制的鸿沟：控制多少线程传输多少数据。

同时我们也决定重新设计一些事件驱动和任务协调类的API来应对日益增长且已经被记录的Reactive扩展。

Reactor是被Pivotal赞助，并且有两名核心贡献者已经被Pivotal雇佣。由于Pivotal同时也是spring框架的东家，我们的很多同事也是不同spring项目的核心贡献者，所以我们也提供从Reactor到spring的集成同时也支持spring框架的一些重要功能如spring消息模块的STOMP代理。也就是说，我们不会强迫仅仅想使用Reactor的去适应spring。我们保留了独立大量的Reactive内嵌工具。事实上，Reactor在的目标之一就是在解决异步和功能性问题时保持中立。

Reactor遵循[Apache 2.0 licensed](http://www.apache.org/licenses/LICENSE-2.0.html)，并且可以在[Github](https://github.com/reactor/reactor)上获取。

