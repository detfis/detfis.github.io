---
layout: dsp_post
title: QA environment - best practices
categories: dajsiepoznac2017 "software architecture" "software engineering" microservices
description: Some tips on how to setup QA environment so it fulfill its purpose 
---

# What QA environment is?

Quality assurance (QA) environment is the place where you test the functionalities of your system before they are going to be shipped to the production environment. Data, hardware and software should simulate the production environment as closely as possible. 

## QA, staging or testing environment?

You might have heard names like **_staging_** or **_testing_** environment, and adding the **_QA_** environment makes it even more confusing. How to differentiate between them? What is the purpose of each of them?

The problem here is that there are no established definitions. I'll explain what I understand by those terms.

**_testing environment_** - this is where you test and validate that user stories are complete. You can use small, confined dataset and as many mocks as needed

**_staging environment_** - this is the environment that is used after development but before production. It's supposed to be stable, should match production environment as closely as possible. In most cases, **_QA environment_** is the same as **_staging environment_**, but in bigger companies, you may have separate staging environment.

In this blog post I'm not going to talk about **_testing environment_** and I'll focus on QA/staging one.

## Does your QA match Production?

This is probably the first question that is being asked anytime a production deployment fails or some bugs are found on the production environment.

Usually, the first thing that is going to be examined is the configuration files. In many cases that might be it - maybe the hostname of some service has changed or one of the port numbers is misconfigured. Those are trivial and simple to fix problems.

The bigger and harder problem might be something else - the mismatch in topology between QA and production. **_Topology_** in that context is the number of servers and connectivity. To give you an example, if your production environment uses a firewall, 2 databases servers and 2 application servers behind a load balancer - this is exactly how your QA environment should look like.

Obviously, the main barrier to making QA's topology exactly the same as production one is cost.

## Zero, One, Many

There is an old saying that the only sensible numbers in the computer science industry are 0, 1 and many. There is a fundamental difference between 1-to-1 and 1-to-many communication. You can't test many scenarios if in your QA env you are using single instance, while in production you run a couple of them. On the other hand, if your production environment uses dozens of servers, it doesn't mean that you have to run dozens in QA - 2 will usually do.

## You play the way you practice

In any sport, the team will play the real game in the same way it practices. This holds true in software development as well. Let's look at the firewall for example - it's extremely hard to take system that is almost completed and identify all the firewall rules that needed to make it work in production. On the other hand - if the development team works with the firewall from the very beginning it's easy to keep track of needed holes in the firewall when new integration points are introduced.

The same applies to QA testing. If your production database has millions of records - your QA database should match it. If you have to use a load balancer in the real system - your QA env should use it as well.

## How not to go bankrupt?

As I mentioned before, the biggest obstacle to having QA topology exactly the same as production one is cost. You don't need to buy exactly the same number of servers for QA that you run in production - one is definitely not enough, but two will be sufficient in most cases. You can lower your costs further, though. If you use some virtualization technologies (like [Virtualbox](https://www.virtualbox.org/) or [VMware](http://www.vmware.com/)) or containerization technologies (like [Docker](https://www.docker.com/)) you can create multiple virtual hosts on a single server. 

## Summary

Today I explained what QA environment is, why it's important to make it as similar to production one as possible and showed you some ways of achieving this. 