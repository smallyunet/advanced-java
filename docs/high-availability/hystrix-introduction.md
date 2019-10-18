## Building highly available service architecture with Hystrix
Reference resources [Hystrix Home](https://github.com/Netflix/Hystrix/wiki#what)。

### What is Hystrix?
In a distributed system, each service may call many other services. The services that are called are **dependent services**. Sometimes it is normal for some dependent services to fail.

Hystrix allows us to control calls between services in a distributed system, adding some **fault tolerance mechanisms** such as **call delay** or **failure dependent**.

Hystrix can prevent a dependent service from spreading in all dependent service calls of the whole system when it fails by **Resource isolation** of the dependent service. At the same time, Hystrix also provides a fallback degradation mechanism when it fails.

All in all, Hystrix helps us improve the availability and stability of distributed systems through these methods.

### History of Hystrix
Hystrix ais a framwork for high availability assurance. The API team of Netflix (which can be regraded as a video website like Youku or iqiyi from abroad) started to do some work to improve the usability and stability of the system in 2011, and Hystrix has developed since then.

In 2012, Hystrix became more mature and stable. In Netflix, besides the API team, many other teams began to use Hystrix.

Today, there are billions of calls between services in Netflix every day, through the hystrix framework, and hystrix also helps Netflix website improve thw overall availability and stability.

[In November 2018, hystrix announced on its GitHub homepage that it will no longer open new features and recommend developers to use other open source projects that are still active](https://github.com/Netflix/Hystrix/blob/master/README.md#hystrix-status). The change of maintenance mode does not mean that Hystrix is no longer valuable. On the contrary, Hystrix has inspired many great ideas and projects. Out highly available knowledge will be explained to hystrix.

### Design principles of Hystrix
- **Control and fault-tolerant protection** for call delay and call failure when calling dependent services.
- In a complex distributed system, it is necessary to prevent a service dependent fault from spreading in the whole system. For example, if one service fails, other services will also fail.
- Support for `fail fast` and fast recovery.
- Provides support for fallback graceful degradation.
- Support near real-time monitoring, alarm and operation and maintenance operations.


Here's a chestnut.

There is a distributed system in which service a depends on service B and service B depends on Service C/D/E. In such a mature system, for example, there may be only 100 thread resources at most. Normally, 40 threads call service C concurrently, and 30 threads call D/E concurrently.

It only takes 20ms to call service C. Now, because Service C fails, such as delay, or hangs, the thread will hang for about 2S. All 40 threads are stuck. Dut to the continuous influx of requests, other threads are also used to call service C. which will also be stuck. In this way, the thread resources of service B are exhausted, unable to receive new requests, and even a large number of therads are running continuously, leading to their own downtime. Service a also hangs up.

![service-invoke-road](/images/service-invoke-road.png)

Hystrix can isolate its resources, such as limiting service B to 40 threads calling service C. When the 40 threads are hung, the other 60 threads can still wok normally. So as to ensure that the whole system will not be dragged down.

### More detailed design principles of Hystrix
- Prevent any dependent service from exhausting all resources, such as all thread resources in Timcat.
- To avoid request queuing and baking, current limiting and `fail fast` are used to control failures.
- Provide fallback degradation mechanism to deal with failures.
- Resource isolation technologies, such as `bulkhead`, `swimlane` and `circuit breaker`, are used to limit the impact of any service dependent failure.
- Through the near real-time statistical/monitoring/alarm funcation, to improve the speed of fault detection.
- Improve the speed of fault handling and recovery through near real-time attribute and configuration **hot modeification** function.
- Protect all failures that depend on service calls, not just network failures.
