# About the Project

The project started in 2012, with a long internal incubation time. Reactor 1.x appeared in 2013. Reactor 1 has been deployed successfully by various organizations, both Open Source (e.g. Meltdown) and Commercial (e.g. Pivotal RTI). In 2014 we started collaborating on the emerging Reactive Streams Standard and started a massive re-engineering targeting April 2015 for version 2.0. The Reactive Streams Standard closed the last gap in our Dispatching mechanism: controlling how much in-flight data was hitting Thread boundaries.

Parallel to that work we also decided to re-align some of our Event-Driven and Task Coordination API to the increasingly popular and documented Reactive Extensions.

Reactor is sponsored by Pivotal where the two core committers are employed. Since Pivotal is also the home of the Spring Framework and many of our colleagues are core committers to the various Spring efforts, we both provide integration support from Reactor to Spring as well as support some important functionality of the Spring Framework like the STOMP broker relay in spring-messaging. That said, we don’t force anyone to adopt Spring just to use Reactor. We remain an embeddable toolkit "for the Reactive masses". In fact one of the goals of Reactor is to stay un-opinionated in the ways you solve asynchronous and functional problems.

Reactor is Apache 2.0 licensed and available on GitHub.


