# 1. Reliability, Scalability and Maintainability


## 1.1. What are data-intensive applications

Many applications nowadays are _data-intensive_, dealing with large amounts of data, complex data or changing data, as oposed to _compute-intensive_ (CPU use).

Data-intensive applications are usually made of standard building blocks:

* __Databases:__ storage and retrieval of data
* __Caches:__ remembering the results of expensive operations
* __Search indexes:__ searching data by keyword or filtering data
* __Stream processing:__ sending messages to other applications, handled asynchronously
* __Batch processing:__ periodic crunch of large amounts of accumulated data

Data-intensive applications may need to use many of these tools to achieve their goals, so your special-purpose data system is made of smaller general-purpose components.

The API usually hides implementation details from clients, the application code takes care of ensuring the smaller components work correctly in unison (i.e. if you update your database, the code should invalidate or update the cache).


## 1.2. What are reliability, scalability and maintainability?

* __Reliability:__ A system should continue to work _correctly_ even when there are software, hardware or human error faults
* __Scalability:__ A system should be able to deal with growth (load, data volume or complexity)
* __Maintainability:__ A system should allow people working on it to do so productively


## 1.3. Reliability

    A system should continue to work _correctly_ even when there are software, hardware or human error faults.

Examples of reliability:

* An application works as expected even if a user makes a mistake or uses the application in unexpected ways
* Peformance is good under the expected load and volume
* The system prevents unauthorised access and abuse

If a system can anticipate faults and handle them, it's called _fault-tolerant_ or _resilient_.

A _fault_ is when one component of the system derives from its spec. This is not the same as a _failure_, which is when the system as a whole stops working.

In order to improve fault-tolerance, it's a good idea to regularly cause faults so that the fault-tolerance is routinely tested and we can predict how the application responds to them. See the [Netflix Chaos Monkey](https://netflix.github.io/chaosmonkey/) as an example.


* __Hardware faults:__ since applications have increasing data volumes and computing demands, more machines are needed which increases the potential for faults. Systems should tolerate the loss of entire machines
* __Software errors:__ such bugs can affect all instances of an applications and they are not quick to resolve. There's many approaches to help with this: testing, monitoring, metrics, analysing assumptions and interactions...
* __Human error:__ they can happen frequently, so to minimise these, the system should minimise opportunities for error, test thoroughly, have quick recovery, monitoring, and provide good user training


## 1.4. Scalability

    A system should be able to deal with growth (load, data volume or complexity).


### 1.4.1. Describing load

_Load_: it can be described with load parameters, which depend on the architecture of the system (Requests Per Second, read/write ratio to a DB, active users, cache hits rate...)

Example of scalability for Twitter's solution to update the timeline of tweets for a user:

* __Solution 1.__ When a user posts a tweet, it's inserted to a global table of tweets (see below for DB example). Queries return the current tweets for the timeline of a user (tweets of people that the user follows).

* __Solution 2.__ There is a cache for each user's timeline. When a user posts a tweet, the cache of each follower is updated to include this tweet. Any requests to read the timeline are fast since the result has been computed ahead of time.

* __Solution 3.__ Hybrid approach: for users with a very large number of followers, solution 1 is used, for the rest of users, solution 2 is used.

Solution 1 was not scalable and caused issues when the application grew in number of users, so Solution 2 helped cope with the growth, making the application more scalable. The reason why the second solution was more performant is that there are way more requests to read the timeline than to post a tweet. The downside is that there is more work required by the application when a user posts a tweet.

In this example, the distribution of followers per user is a load parameter to consider when discussing scalability.


### 1.4.2. Describing performance

Once load has been described, we investigate whan happens when load increases:

- Increase one of the load parameters (i.e. requests) and keep everything the same, what happens?
- How much do you need to increase resources to when the load parameter is increased so that performance is not affected?

Performance can be described in different ways:

* For a batch processing system (offline) it's important to consider how many transactions can be processed per second, the _throughput_.
* For an online system, it's important to consider the response time of a service. This is usually reported based on percentiles, since using an average value will be impacted by outliers.

    __Note:__
    - _Response time_ would be the time it takes for the response of a request to reach a client (the actual time it takes to process, plus network delays, queues, etc).
    - The _latency_ is the time the request is waiting to be handled.
    - The _service time_ is the time it takes to process a request.
 
It's important to measure response times on the client side to understand how long it takes for a request to reach a client, taking into consideration the latency as well.
A single slow request in a server can slow down an entire user request if many requests are needed to serve it.


### 1.4.3. Coping with load

We can _scale up_ (vertical scaling - more resources in a machine) or _scale out_ (horizontal scaling - distribute the load in more smaller machines or instances).

In most situations, you need a hybrid approach where you have multiple powerful machines, rather than only a few very powerful ones or a large number of small ones. There is no universal solution and this would depend on the load parameters for the application.

Some systems can cope with increasing load in an automated way, scaling as needed (_elastic_ systems). Otherwise, this can be done manually, it's more time consuming but the system is simpler.


## 1.5. Maintainability

    A system should allow people working on it to do so productively.

The majority of the cost of software is not in it's initial development but in its maintenance. We will spend the majority of the development time maintaining an application (fixing it, adding new functionality, tech debt...), so it's important to work in a way that reduces the effort needed to do this work, avoiding the creation of legacy software.

There are three principles for this:

* __Operability:__ make it easy for operations to run smoothly
* __Simplicity:__ make it easy for engineers to understand the system, remove complexity
* __Evolvability:__ make it easy for engineers to make changes in the future (AKA extensibility, modifiability or plasticity)


### 1.5.1. Operability

We can make operations easier by:

- Giving visibility into runtime behaviour, such as with monitoring tools
- Providing support for automation and integration with tools
- Making the application resilient so it can tolerate machines being down (don't run on individual machine)
- Providing good documentation
- Providing good default behaviour with the ability to override these defaults
- Self-healing with the ability for admins to take control if needed
- Predictable behaviour


### 1.5.2. Simplicity

