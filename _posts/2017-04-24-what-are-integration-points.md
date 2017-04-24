---
layout: dsp_post
title: Microservices - what are integration points?
categories: dajsiepoznac2017 "software architecture" "software engineering" microservices
description: What are integration points, why they are an unavoidable threat to the stability of your system and how to handle them gracefully
---

# What are integration points?

Recently I've read [Release It!Design and Deploy Production-Ready Software](https://pragprog.com/book/mnee/release-it) - one of the best, no-bullshit and hands-on books on software architecture I've ever read. I highly recommend you to read it - this is a true repository of knowledge and a really good read. 

One of the concepts introduced there was an **_integration point_**. An **_integration point_** is every single place where your application talks to the outside world. It can a payment processing provider for your online shop, an authentication provider if you're using [OAuth](https://oauth.net/2/) or it can be a cron job that runs daily and populates your database with the latest stock market information - in fact it can be any third party API you rely on. 

But **_integration points_** are not restricted to third party services. In fact, it can be any remote procedure call and in the era of microservices, we incorporate more and more of them to our applications. 

Let's say you're not building a complex, distributed system. Your job is to maintain a company blog - a simple web app that provides a possibility to publish and edit posts. Even in this simple scenario, you're not free from **_integration points_** - you probably store some data in the database, and database connection is a legit **_integration point_** as well.

As you can see, **_integration points_** are inherent parts of every application, even in the very simple ones you can find a couple of them, and if the complexity of your application grows - the number of integration points can be measured in dozens or even hundreds.

## Third-party APIs

As I mentioned before, when we talk about **_integration points_** the most obvious example is when we integrate with some services - for example, we want to send a file to an S3 bucket or want to use Google Maps API - there are thousands of possibilities. The common mistake we do, especially when you integrate with any of the well-known APIs, is the assumption that when we make a request, we are always going to get some results back. We push responsibility for our system on the authors and maintainers of the service and we mistakenly belief that since the service is used by many users and supported by a big company - it's definitely 100% available and does not have any bugs. 

This is obviously false - none service is available 100% of the time and the authors of the service are just developers, who make mistakes like any other developer in the world. You have to be prepared that those services will not be responsive. To be honest, You should go one step further and you should **assume** that sooner or later they will crash and make sure they will not take your system down.

## Problems with integration points

**_Integration points_** are the biggest threat to the stability of your system. You want your system to talk to other services, save something on the disk or to the database, etc - so those threats are unavoidable and they are present in every single system. Our job, as the software engineers, is to mitigate those risks. 

**_Integration points_** comes in many flavors, so the nature of the possible failures will differ:
- if you log something and save it on the disk you may end up not having sufficient rights to write on that disk, or you may run out of the space on this device
- if you want to call some API, you may not have an internet connection, the API may not be available or specification can change and your call is no longer correct
- when you use a database you may end up waiting for a connection from the pool because some threads has been blocked

The blocked threads are especially evil - it's relatively easy to track when and why your application has crashed, but blocked threads can gradually lead to slower response times, which can lead to the crash of the system. In fact, for the end user there is no difference whether your system is not working or is working terribly slow - if he can't get a response quickly enough, he would leave. 

The next problem with **_integration points_** is something called **_cascading failures_**. **_Cascading failures_** happen when the problem in one layer or part of the systems causes problems in callers. The simplest scenario might be when a database is down and the rest of the application can't handle this properly and the end user sees error stack trace on their screen.

## How to handle integrations points?

The prevalent solutions to the problems caused by **_integration points_** are **_circuit breakers_**. A circuit breaker is a design pattern that allows you to make sure that you are not making unnecessary requests to a system which is not responding. When the receiver is working correctly, the circuit breaker is open. Once requests to the receiver fail, the circuit breaker closes. Subsequent calls to the receiver are not attempted and an error message is returned immediately, without wasting resources to reach the system that we are sure is not working now. After a given period of time requests are allowed to go through, to check whether normal workflow could be resumed. If not, the circuit breaker will keep blocking the calls. You can read about them more [here](https://martinfowler.com/bliki/CircuitBreaker.html).

Apart from introducing **_circuit breakers_** to your system you should provide monitoring mechanism to have visibility into which **_circuit breakers_** are open and which are closed. I'm planning to write a detailed blog post on this topic soon. 

Another pattern that complements **_circuit breaker_** is **_timeout_**. **_Timeouts_** prevent operations from hanging forever. Request that waits for a very long time are using up resources, like thread pools, and may result in **_cascading failures_**. **_Timeout_** let's know **_circuit breaker_** that the receiver is unavailable now and there is no point making calls there now.  

## Summary

In this post I've introduced to you the problem that **_integration points_** entails and how you can counter then using **circuit breaker** and **_timeout_** patterns. I'm planning to write more about good practices in software architecture soon and I hope you've found this post informative and helpful.
