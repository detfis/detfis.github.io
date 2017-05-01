---
layout: dsp_post
title: Fail Fast Principle
categories: dajsiepoznac2017 "software architecture" "software engineering" microservices
description: Improving stability of your system by applying Fail Fast principle
---

# Fail Fast Principle

In [one of the previous posts]({{ site.url }}/what-are-integration-points/) I talked about **_circut breaker_** and **_timeout_** patterns as a way of improving stability of your application. In this blog post I will talk about a pattern that compliments the previous ones - **_Fail Fast_**.

This patterns is also taken from the excellent [Release It!Design and Deploy Production-Ready Software](https://pragprog.com/book/mnee/release-it) book and I recommend that you will read it.

It is very simple principle and it's based on common sense - if the system can determine in advance that it won't be able to succeed, it's always better to fail fast.

Thanks to this you don't have block any resources and those resources can be used to complete more meaningful job.

## How to apply Fail Fast Principle

It doesn't mean that your system has to introduce some fancy machine learning algorithms to figure out if the request will succeed.

It's actually much simpler than this - you can for example check if all **_integration points_** needed for this tasks are working properly. You can do this by verifying the state of **_circut breakers_** for those **_integration points_**. 

Another way to fail fast is performing basic parameter checking before calling actual action. You can verify up front if all needed parameters are present or if they are in correct format. Word of warning here - make sure you do only sanity check type validation here. Any business logic specific validation should belong to the some kind of domain specific object - otherwise the rule encapsulation would be violated.

## Reporting a system failure

Import thing when you apply **_Fail Fast Principle_** is correct failure reporting. You should make sure that system failures (database or network is unavailable, some external service is down etc) are reported differently than application failures (when user submits invalid parameters). In particular, invalid input should never change the state of any **_circut breaker_**.

## How does it work with Circut Breakers and Timeouts?

As I mentioned before - **_Fail Fast_** works as a compliment of **_Circut Breaker_** and **_Timeout_** patterns. Both helps you deal prevent cascading failures. **_Circut Breaker_** and **_Timeout_** deal with incoming requests by giving up when appropriate. For incoming requests you should **_Fail Fast_** if you are not able to process the request.

## Summary

**_Fail Fast_** is a great pattern that can significantly improve stability of your system by preventing long responses. Long responses can lead to cascading failures, which is the worst scenario for any application. They can also improve capacity of your system by avoiding using up resources on tasks that won't succeed anyway