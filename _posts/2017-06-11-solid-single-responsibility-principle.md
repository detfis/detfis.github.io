---
layout: post
title: SOLID - Single Responsibility Principle
categories: solid "software architecture" "software engineering"
description: What is Single Responsibility Principle, why it is so important and how to apply it
---

# Single Responsibility Principle

With this blog post, I'm starting a series of posts that will describe every single [SOLID](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)) principle. In those posts, I will give you a definition of the rule, why it's useful, how to apply it and I'll give you examples of the code that break and obey this particular rule. My goal is, that after every post you will have a strong understanding of every principle, you will be able to determine whether a given piece of code complies with it and will be able to use it in your own project.

## What is SOLID?

SOLID is an acronym coined by Michael Feathers for the five basic principles of object-oriented design introduced by [Robert C.Martin](https://en.wikipedia.org/wiki/Robert_Cecil_Martin), known in the IT community as Uncle Bob in the early 2000s.

Those principles are:
* S - Single Responsibility Principle
* O - Open/Closed Principle
* L - Liskov Substitution Principle
* I - Interface Segregation Principle
* D - Dependency Inversion Principle

Why is it relevant for you? Should you care about software design principles invented almost 20 years ago? Definitely YES! The premise of this principles is that if you follow them, they will guide you towards a good design of your system. This is very useful, because very often when you start coding you don't know how to structure your application, where to put given a piece of code or functionality, and [SOLID](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)) principles "reveal" to you proper design of your system. 

What is a "good design", you may ask? It's a very good question - since "good design" is a very elusive term. Probably every software developer has its own definition of it. I'll share with you mine - good design gives you 2 main things:

* describes what an application is doing - code is definitely often read than written, so it should be easy to figure out what flow of the application is, what are main functionalities and it should be relatively easy to find code responsible for given business logic

* embraces the change - your application is rarely "done". New feature requests are coming in, bugs need to be fixed, sometimes you need to deal with scaling issues etc. It's certain, that you will be asked to change something with your existing code base. You should design your system to keep as many doors open as long as possible and to make those unavoidable changes as cheap as possible.

## What is Single Responsibility Principle

I'm not going to focus on a formal definition of this rule but if you're interested in it, you can find it [here](https://en.wikipedia.org/wiki/Single_responsibility_principle). What I would like to achieve is to help you develop an intuitive understanding of this principle. For me, the most helpful explanation if SRP was this:

* a class/module/function does only 1 useful thing
* a class/module/function has only 1 reason to change

That's it. It's that simple and you have to admit that it just a common sense. Following this rule will have a profound impact on your design, though. Introducing this fix many of the issues you have with your system. It will make your classes more cohesive since responsibilities will be separated. You will have to put more effort into thinking what this particular class is supposed to do and as a result, you will end up having a well-defined behavior of each class. As a side effect, your classes and functions will have better names what will improve readability of the system. Naming, in general will be better, since now you know exactly what this piece of software is supposed to do.

The biggest advantage of applying SRP in your codebase is to increase reusability of your components. Let's say, that after refactoring your code you have a class that is responsible for sending email when a user has registered. Your new feature request is to send an email when a user wants to reset his password. You're in great position to take an email sending component and use it for sending new kind of email! You can actually prepare it so it is totally unaware what type of email it sends so you can use it anywhere you have to send a message.

## How to determine if your class comply with Single Responsibility Principle?

Sometimes, especially at the beginning of using SRP, it might not be clear if your class of function is in line with this principle. Even if you're an experienced developer, if you work on a complex problem or in complicated domain, it's very easy to break SRP by putting too much responsibility within one class or function. 

There is a simple way of checking if your class does only 1 thing - describe in 1 sentence what this class or function is doing. If you can't do this, it has definitely too many responsibilities. There are some caveats to be aware of:

* if you're saying: _"my class does this **AND** this"_, it means it has 2 responsibilities instead of one 
* if you're saying: _"my class does this **OR** this"_, it's even worse, because your class has 2 responsibilities which aren't related to each other

Sandi Metz in her book [Practical Object-Oriented Design in Ruby](http://www.poodr.com/), which I highly recommend reading even if you're not Ruby developer, describes a great way of checking if this particular function belongs to correct class. Let's say you have a class `User` which has a function that returns his full name, address, and content of email when a user requests a password change. 

To determine if this function belongs to the correct class you have to ask a _class name_ to do for you, what this _function_ is supposed to do. It may not be clear when I describe it, but after seeing a couple of examples you will get what I mean:

_"User, what is my full name/address?"_ makes complete sense and indicates that those methods belong to _User_ class.

_"User, what is the content password reset email"_ is not so clear and you can start wonder if User class is correct place for this function.

## How to make your code SRP-compliant?

The basic technique to separate responsibilities among class is just to extract extra behavior to a new class or move it to the more appropriate existing one. The same applies on the function level - if your function does to many things, create another function and use it in the original one.
The common mistake is to mix iterator with the actual function logic. Imagine, you want to mark a user as _inactive_ if they did not log in withing last 2 months. The pseudo code for this might look like that:

```
function mark_users_as_inactive
  for each active user
    if user has not logged in within 2 months
      user.inactive = true
      user.save
``` 

As you can see, we mix here logic of fetching a number of users and iterating over them with the actual logic of marking a user as _inactive_. We can refactor this code a little:

```
function mark_users_as_inactive
  for each active user
    mark_as_inactive(user)
      
function mark_as_inactive(user)
  if user has not logged in within 2 months
    user.inactive = true
    user.save
```

## Summary

That's all for today's post on Single Responsibility Principle. I hope I helped you understand what it is and how and why you should apply it to your project. If you found this post useful or something is not clear for you - leave a comment below and I'll answer it as soon as possible.
