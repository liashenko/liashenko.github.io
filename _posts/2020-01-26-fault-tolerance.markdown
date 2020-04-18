---
layout: post
title:  "Fault-tolerance in distributed systems"
date:   2020-01-28 21:25:00 +0200
permalink:   fault-tolerance
categories: [architecture, distributed systems]
tags: [architecture, dystributed systems, fault tolerance, hystrix, chaos engineering]
---

<center><img src="/assets/posts/resilient-systems/service-and-dependecies1.jpg" style="max-width:500px"></center>   
> A distributed system is a network of computers, which are communicating with each other by passing messages, but acting as a single computer to the end-user.

With distributed power comes big challenges, and one of them is inevitable failures caused by distributed nature.    
Network connections fail or degrade, servers crash or respond enormously slow, software has bugs, etc.  

How to make your system stable and tolerant to the failures?
* **Make your components redundant**. Avoid single point of failure.
* **Handle your Interaction Points** (calls to remote services).
* When it's possible, **respond to requests when faillures happen**.
* **Test your system** to discover its behavior under pressure.
* Embrace the chaos to bring order in your system facilitating **Chaos Engineering** experiments.

### Handle your Interaction Points
> Integration points are the number-one killer of systems.
  
Every remote call is a risk to your system health and a single failing call can take the whole system down if not handled properly.  
Let's review some common patterns to handle remote calls.   
<center><img src="/assets/posts/resilient-systems/failed-deps.jpg" style="max-width:500px"></center>    

#### Retries
Often trying the same request again causes the request to succeed. It happens because of partial or transient failures.  
A partial failure is when a part of requests succeed.   
A transient failure is when a request fails for a short period of time.  

<center><img src="/assets/posts/resilient-systems/retry.jpg" style="max-width:400px"></center>     
But it's not always safe to retry. A retry can increase the load on the system being called. Instead of retrying immediately, you can use **exponential backoff**, where the wait time is increased exponentially after every attempt.  
```
waitTime = min(maxWait, baseInterval * exponentialFactor ** attempt)
```
When failures are caused by overload, backing off doesn't help. If all the failed calls back off at the same time, they increase overload even more.  
The solution is **jitter**. Jitter adds randomness to the backoff to spread the retries in time. 
```
waitTime = rand(0, min(maxWait, baseInterval * exponentialFactor ** attempt))
``` 

#### Timeouts 
When a request is taking longer than usual, it might increase latency in your system (and fail eventually).  
Also, the call holds on to the resources it is using for that request and during high load the server can quickly run out of the resources (memory, threads, connections, etc.).

To avoid this situation set **connection and request timeouts**.  

<center><img src="/assets/posts/resilient-systems/timeouts.jpg" style="max-width:450px"></center>  

#### Circuit breakers
When there’s an issue with a dependency, stop calling it!  
In the normal “closed” state, the circuit breaker executes requests as usual.   
Once the number of failures for the frequency of failures exceeds a threshold, the circuit breaker “opens” the circuit for some time.  

<center><img src="/assets/posts/resilient-systems/cb.jpg" style="max-width:450px"></center>  
#### Bulkheads
> In a ship, a bulkhead is a dividing wall or barrier between other compartments.  
> If the hull of a ship is compromised, only the damaged section fills with water, which prevents the ship from sinking.

Isolate the failure.    
Separate thread pools dedicated to different functions (e.g. separate thread pools for each remote service), so that if one fails, the others will continue to function.   

<center><img src="/assets/posts/resilient-systems/bulkheads.jpg" style="max-width:450px"></center>   

### Respond when failure happens
“Fail fast” is generally a good idea:
* no increased latency
* no risk for the whole system to halt
* no invalid system behaviour
* releasing the pressure on underlying systems (i.e. shed load) when they are having issues 

However, there are scenarios where your service can provide responses in a “fallback mode” to reduce the impact of failure on users.  

**Cache**  
Save the data that comes from remote services to a local or remote cache and reuse the cached data as a response during one of the service failure.
<center><img src="/assets/posts/resilient-systems/cache.jpg" style="max-width:350px"></center>
**Queue**  
Setup a queue for the requests to a remote service to be persisted until the dependency is available.   
<center><img src="/assets/posts/resilient-systems/queue.jpg" style="max-width:400px"></center>
**Stubbed (default) values**  
Return default values when personalized options can’t be retrieved.  
**Fail silently**.  
Return empty or null response that can be handled by the caller (e.g. UI).    
If possible, disable the functionality that is failing.

### Hystrix
[Hystrix](https://github.com/Netflix/Hystrix/) is a Netflix open-source library that helps you handle Integration Points using the techniques described before: Timeout, Circuit Breaker, Bulkhead, and without effort allows you to provide fallback options.   
[<img src="/assets/posts/resilient-systems/hystrix-command-flow-chart.jpg">](../assets/posts/resilient-systems/hystrix-command-flow-chart.jpg)   

Embed the fault tolerance and latency tolerance in your system wrapping the calls to external services into HystrixCommands:   
<center><img src="/assets/posts/resilient-systems/hystrix-service.jpg" style="max-width:600px"></center>

### Testing
#### Load testing and stress testing
Perform load and stress testing to discover how your system behaves under the load. It might uncover unexpected issues and failures in your system.  
Perform the testing for the long period of time to discover how your system behaves under continuous stress.
<center><img src="/assets/posts/resilient-systems/load-testing.jpg" style="max-width:500px"></center>

#### Test for remote services failures
* no response
* failed response
* slow response

#### Chaos engineering (resilience testing)
> Chaos Engineering is the discipline of experimenting on a system in order to build confidence in the system’s capability to withstand turbulent conditions in production.

Facilitate Chaos Engineering experiments to understand the system robustness and discover the system weaknesses.  

[Chaos Engineering experiments](http://principlesofchaos.org/) follow four steps:
> 1. Start by defining ‘steady state’ as some measurable output of a system that indicates normal behavior.
> 2. Hypothesize that this steady state will continue in both the control group and the experimental group.
> 3. Introduce variables that reflect real world events like servers that crash, hard drives that malfunction, network connections that are severed, etc.
> 4. Try to disprove the hypothesis by looking for a difference in steady state between the control group and the experimental group.

#### Gremlin
[Gremlin](https://www.gremlin.com) is a chaos engineering platform. Gremlin provides the framework to safely and simply simulate real outages.

Be prepared - Gremlins come:  
* Resource gremlins. Throttle CPU, Memory, I/O, and Disk
* State gremlins. Reboot hosts, kill processes, travel in time
* Network gremlins. Introduce latency, blackhole traffic, lose packets, fail DNS
 
### Conclusion
In distributed systems failures are unavoidable by nature. Keep that in mind during architecture, implementation and testing of your system.   
Handle the Integration points with Retries, Timeouts, Bulkheads and Circuit breakers.  
Minimize failures impact on users by responding when failures happen. Leverage Caching, Queues, return default values or disable the failing functionality.  
Test your system vigorously. Test for remote services failures.   
Break your system to make it unbreakable facilitating chaos engineering experiments.  

### References
1. Michael T. Nygard. Release It!: Design and Deploy Production-Ready Software 2nd Edition (2018)
2. [Article "Fault-tolerance in a high volume distributed system" by Netflix](https://netflixtechblog.com/fault-tolerance-in-a-high-volume-distributed-system-91ab4faae74a){:target="_blank"}
3. [Article "Lessons learned from the AWS outage" by Netflix](https://netflixtechblog.com/lessons-netflix-learned-from-the-aws-outage-deefe5fd0c04){:target="_blank"}
4. [Article "Exponential backoff and jitter" by AWS](https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/)
5. [Hystrix](https://github.com/Netflix/Hystrix/wiki){:target="_blank"}
6. [Principles of chaos](http://principlesofchaos.org/){:target="_blank"}
7. [Gremlin](https://www.gremlin.com/){:target="_blank"}