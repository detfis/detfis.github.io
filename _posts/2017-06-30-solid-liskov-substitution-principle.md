---
layout: post
title: SOLID - Liskov Substitution Principle
categories: solid "software architecture" "software engineering"
description: Explanation in simple words what the Liskov Substitution Principle is.
---

# Liskov Substitution Principle

[Liskov Substitution Principle](https://en.wikipedia.org/wiki/Liskov_substitution_principle) represents the letter *L* in the [SOLID](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)) acronym. In my personal opinion, it's the trickiest and the most subtle principle out of those five. It can be hard to understand, sometimes a little counter-intuitive and it's not always obvious when you break this rule.

LSP(Liskov Substitution Principle) talks about inheritance. It states that you should be able to replace any instance of the parent class with any instance of its subtypes without introducing any unexpected or incorrect behaviour. 

There is nothing surprising here, right? Inheritance answers the question "IS-A" - "Dog IS-An Animal", "Car IS-A Vehicle" etc, so it's quite clear that we should be able to exchange those objects in our code - this is the reason why we used inheritance in the first place.

Unfortunately, if we want our code to be [SOLID](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design))-compliant we need to think harder about our inheritance structure.

## Examples of breaking the Liskov Substitution Principle

Firstly, I'll show you a classic example of breaking LSP rule. It's a good one because it shows how counter-intuitive inheritance can be and how much thought we need to give to make it correct. 

This example is about _**Rectangles**_ and _**Squares**_. From the math class, you should know, that "square IS-A rectangle", so when you are supposed to model this in your code it's very natural to use inheritance here - it answers YES to the "IS-A" question and since the first geometry class at school are thought that this is true.

Let's create a very simple implementation of this. 

```ruby
class Rectangle

  def initialize(width, height)
    @width = width
    @height = height
  end

  def width=(new_width)
    @width = new_width
  end

  def height=(new_height)
    @height = new_height
  end
end

class Square < Rectangle
  def initialize(length)
    super(length, length)
  end
end
```

So far so good, why might think that we are done here. But there is some unexpected and incorrect behaviour hidden in our code. When you change the height of the square you would expect that the width changed as well, which is not true in the case of a parent class, the rectangle.

It's confusing, isn't it? We directly translated our math knowledge into the code and now we have buggy code. How can we explain this?

This is where it gets tricky. [Uncle Bob]((https://en.wikipedia.org/wiki/Robert_Cecil_Martin)) explain this in this way:

```
The problem is this is not a Rectangle. It’s a piece of code! It’s a representation of a Rectangle. Representatives do not share the relationships of the things they represent. For example, imagine 2 people who are getting divorced. Each one of them has a lawyer who represents them. It's very unlikely those 2 lawyers themselves are getting divorces because the representatives of things do not share the relationships of the things they represent!
```

Let's look at another example:

```ruby
class Car
  def fuel_up
    # implementation
  end
end

class ToyCar < Car
  def fuel_up
    ???
  end
end
```

Here we might be tempted to model the toy car as the kind of the car, but as we see we will have problems implementing almost every method from the parent class in the subclass. This gives us a strong indication that we have used the wrong abstraction here.


## Violations of the Liskov Substitution Principle

We will look now at the common violations of the LSP. I've seen them many times and very often they are not even considered a bad design, but in fact they are constructed incorrectly.

The first code smell is using the _**kind of**_ expression in the parent class and branching out based on this. It clearly shows that you have the wrong abstraction and you should revisit your design.

Another glaring example is raising new exceptions in the subclasses. I'll give you an example - let's say we have parent class named _**Car**_ and it's subclass - _**TeslaCar**_ and one of the Car's method is _**shift_up_gear**_. Obviously, _**TeslaCar**_ does not have gear, so we can implement this raising exception in the _**TeslaCar**_ class:

```ruby
class Car
  ....
  def shift_up_gear
    # implementation goes here
  end
end

class TeslaCar < Car
  def shift_up_gear
    raise "Not Implemented"
  end
end
```
As we can see, we introduced an exception to the subclass, so we violated the LSP.

Another, this time less obvious way of breaking LSP is introducing additional behaviour to our method. Let's look at the rectangle and square example again. One of the possible implementation to address the issue with changing height without changing width in the square class might be this:

```ruby
class Square < Rectangle

  def width=(new_width)
    @width=new_width
    @height = new_width if @height != new_width
  end

  def height=(new_height)
    @height=new_height
    @width = new_height if @width != new_height
  end
end

```

This should work - our square will have correct values, but since we are changing width along with the height, which happens only in the subclass and not in the parent class - we are violating the Liskov Substitution Principle again.

Last, an even more unnoticeable way of breaking the LSP is messing with pre-conditions or post-conditions. Assume, that one of the methods of our parent class operates on the integers, but our subclass method assumes that those integers are bigger than 0 - in that case our code does not comply with LSP.

An example of breaking LSP because of post-conditions might be closing TCP connections in the parent class method, but leaving them for further use in the subclass method.

## Summary

In this blog post, I brought the concept of Liskov Substitution Principle and it's aftermath to you. As you saw, Liskov Substitution Principle can be difficult to apply correctly. The key takeaway is when you use inheritance you should not only make sure the subclass answers the "IS-A" question, but also that the whole interface of the parent class can be rightly implemented in the subclass.