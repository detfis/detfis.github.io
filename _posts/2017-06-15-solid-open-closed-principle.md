---
layout: post
title: SOLID - Open / Closed Principle
categories: solid "software architecture" "software engineering"
description: What is Open / Closed Principle, why does it matter and how to apply it?
---

# Open / Closed Principle

This is the second post on [SOLID](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)) principles series and today we will talk about Open / Closed Principle. This rule was introduced in 1988 by [Bertrand Meyer](https://en.wikipedia.org/wiki/Bertrand_Meyer), who is also the author of the [Eiffel programming language](https://en.wikipedia.org/wiki/Eiffel_(programming_language)) and the idea of [design by contract](https://en.wikipedia.org/wiki/Design_by_contract). It means that any software entity, like classes, modules or functions, should be open for extension and closed for modification. In other words, we should be able to add new features without the necessity of changing the existing code.

## Practical example

"Code should be open for extension and closed for modification" sounds a little cryptic. When I was learning about this I wasn't able to fully grasp it. It's very simple principle and I believe that showing you an example is the best way of explaining it.

Let's imagine we are are generating some kinds of reports in our application. For now, we are generating XML and HTML reports, but we got a new requirement that PDF version is needed as well.

The naive approach to implementing this will be having a _**ReportGenerator**_ class where something like this could be found:

```ruby
if :xml
  generate_xml_report
elsif :html
  generate_html_report
else
  ....
```

To add a PDF version we would have to add a new _elsif_ block to the existing class. This design does not comply with the OCP (open/closed principle) because we are forced to change existing class in order to add new features.

How can we make this code better? What are the techniques to make this code open for extension and closed for modification?

## Inheritance

Historically the solution to this problem was to use inheritance. We would have basic _**Report**_ class and the _**XMLReport**_ and _**HTMLReport**_ classes that inherit from it. If we wanted to add PDF report, we would create another subclass of a _**Report**_ class called _**PDFReport**_. 
We achieved our goal here - we added a new feature and we didn't have to change existing code. 

This way of solving things in not actually recommended anymore. Inheritance entails some consequences and we should be very conservative using it.

## What are the drawbacks of inheritance?

The biggest threat of overusing inheritance is having a lot of layers of abstractions that hampers transparency. Subclasses are tightly coupled with the superclass so the flexibility of the code decreases. Some modern languages, like [GO](https://golang.org/), gave up inheritance completely and they rely on type composition exclusively.

## Object composition

More flexible way of making our code OCP-compliant without the drawbacks of inheritance is object composition. There is a design pattern that solves this problem and is called [Strategy](https://en.wikipedia.org/wiki/Strategy_pattern). In this pattern, we encapsulate particular algorithm within a separate class. Client can choose any of the strategies available.

In our case we will have only one _*Report*_ class and a couple of strategies:
- _**HTMLFormatter**_
- _**XMLFormatter**_
- _**PDFFormatter**_
- etc.

To generate a specific type of report we will have pass strategy we want to use to the _**Report**_ class (this is called [Dependency Injection](https://en.wikipedia.org/wiki/Dependency_injection)) and call a _**generate_report**_ method. The code might look like this:

```ruby
class HTLMFormatter
  def generate
    generate_html_report
  end
end

class XMLFormatter
  def generate
    generate_xml_report
  end
end

class PDFFormatter
  def generate
    generate_pdf_report
  end
end

class Report
  def initialize(strategy)
    @strategy = strategy
  end

  def genetate_report
    @strategy.generate
  end
end

Report.new(HTMLFormatter.new).generate_report
```

## Summary

The way of generating report is decoupled from the report itself, so now we can add new kinds of reports without changing the existing code. It means that we are honouring the Open / Closed Principle.
