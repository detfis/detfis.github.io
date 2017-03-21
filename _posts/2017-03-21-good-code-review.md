---
layout: dsp_post
title: Good code review
categories: dajsiepoznac2017 
description: How to do great code reviews for fun and profit.
---

# What is a code review?

According to [Wikipedia](https://en.wikipedia.org/wiki/Code_review), __"code review is a systematic examination (sometimes referred to as peer review) of computer source code"__. 
In simple words - every new piece of code, before it goes to testing and then to staging and then to production should be examined by another team member. We can go to next stage, usually testing, only when the reviewer approved given piece of work.

## Why to bother?

On the first glance, it seems like total waste of time - we're wasting time another developer to read and comment code of his colleague and we still have to go through the normal process, like testing etc, so no visible advantages there. It will be easier to accept and understand when the code of only junior developers would be reviewed, but it's not true - even the work of the most senior developers on the team should be examined.

But trust me, there are deep and profound benefits to this technique, on many levels. I'll try to list just the couple of them:
- {:.margin_bottom} we can find bugs earlier in the process. Whenever we do this, the cost of bug is lower
- {:.margin_bottom} we end up having better code. Just by having only one person to fully understand the code before it goes to the next stage of software delivery process, prevents many kinds of bugs and improves readability, maintainability, understandability, and architecture of the software
- {:.margin_bottom} more people know this part of code deeply. It helps to spread knowledge about the system through the team. Very often people get caught up in very specific part of the code and thanks to reviews they can to learn more about another area of the system. It could help them to get a bigger picture of the product they are building.
- {:.margin_bottom} education of less experienced programmers. Firstly, juniors can look at the way more experienced programmers approach given problem. They can learn how to structure code effectively, how and what to test and much more. On the other hand, they can get feedback about their code - what should they work on, what is not clear, what can be done better or where they improved since the last code review, which can be a great motivation boost. 
- {:.margin_bottom} introduction to the system for new joiners. From my personal experience doing code reviews in the new project is the best and fastest way of getting your head around the code-base. It can take some more time, especially at the beginning, but an understanding of the system comes really fast. Moreover, when your code is reviewed, your colleagues can show you which parts of the project you don't comprehend fully yet. 
- {:.margin_bottom} it improves communication about the code. Code review gives you a great platform to talk about the code, it introduces the process where discussing the code is one of your daily routines.

## Potential issues

Although code reviews have a lot of benefits they have can introduce some issues. None of them is ingrained in code reviews per se, but be aware of their existence and take an action whenever you spot any of them.

First and most common is the fact, that __programmer's ego could be very delicate__. People get attached to their code and sometimes they don't take feedback well. Sometimes they treat it as an attack or undermining their abilities as a software developer. Another reason why the tension between programmers can occur is the inability of the reviewer to give constructive feedback - people sometimes forget that code review is a cooperation tool to improve code quality, decrease an amount of bugs and broaden understanding of the system - not to find the sloppiest developer within the team. 
To prevent this mindset to spread across your company take care of positive attitude around code review. Make sure that everyone understands the goal of code review and teach your colleagues how to give valuable feedback without getting personal or offending anyone. 

The second common problem that happens quite often is __time pressure__. When the deadline of an important feature is coming or your boss is not happy with a number of features your team had delivered last sprint/month/year code reviews could be the easy pick to "streamline" the process. Unfortunately, there is no easy answer to this problem. The only thing you can do is collect metrics - measure amount of bugs that goes to production and compare it to the same metric when you do not do code reviews - the difference should be staggering and it will be a hard proof for the management that peer code reviews actually save them some money.

## How to do it?

Now you know the benefits and potential problems of code reviews, so the next question is - how to do them to be as effective as possible and to deliver the maximum value from the process to the team?

I'll share with you my approach to code reviews. I have to note that I build mostly web applications and this process works fine in this realm. I'm pretty sure that those are general advices and should work just fine in any software industry. Obviously, it may need some tweaks for you, but this is exactly what you should be doing anyway - iterate over the process, find out what worked for you, what did not and improve where you can.

Before starting a code review I want to read and understand the requirements - I want to read them before looking at the code not to get biased by the actual implementation. Next, I usually start with the general overview of the code. I want to make sure that I understand what the code is doing and I look for obvious mistakes - typos, bad variable or function names, commented code left undeleted etc. Nothing too complex, just to get acquainted with this piece of code and to find glaring errors. 

After that I do one of the 2 things - I ask the author to walk me through the code or I start the more in-depth review straightaway. I want a face-to-face explanation when I don't fully understand the code - for example when this is a new project for me or when the domain of the code I review is particularly difficult.

Next, I start doing the thorough review. This is the longest part of code review. When I do this I focus on things like this:
- {:.margin_bottom} Is there a more idiomatic way of solving the problem at hand?
- {:.margin_bottom} Is the code coverage adequate? Or maybe is there better way of testing this functionality?
- {:.margin_bottom} I look through potential refactoring options that could improve readability and maintainability
- {:.margin_bottom} Are the edge-cases covered? Are there any situations where this code could produce wrong results?
- {:.margin_bottom} Does the code comply with the requirements? Are there any logical errors?
- {:.margin_bottom} I try to figure out if the efficiency of the code is good enough or could be improved
- {:.margin_bottom} Did I expect some code to be found in the code review but I couldn't find it? Are there any omissions?

This is a general overview of my process and how I approach code reviews. It works for me, your process might be completely different - might be shorter, might include more people, or maybe face-to-face conversation are not feasible. Anyway - I think this could be a good starting point and from here you can figure out what will work for you.

## Tips

Everyone team should come up with the most effective way of doing the code reviews by themselves, but there are some general best practices that you can incorporate.

Review for at most 1 hour at a time
: {:.margin_bottom} Reviewing requires a lot of focus and uses loads of mental energy. This is why if you want to be productive and want your code reviews to be as good as possible you should refrain yourself from spending long hours doing only code reviews.  

Take your time
: {:.margin_bottom} Don't rush. Sometimes you will need to read the code couple of time until you fully grasp it. Sometimes after reading the code, you will know that something is wrong there, but you could need some more time to identify what exactly could be improved. 

Look for omissions as well
: {:.margin_bottom} This is probably the hardest part of code review - figuring out what should be there but it's not present. Checklists could be good solutions here - for example, they can remind the reviewer to check if unit tests are created, a situation where invalid data has been passed to the function etc.

Limit LOC in code review
: {:.margin_bottom} Do not review huge chunks of code at once. Limit yourself to 400 LOC. That should be the team-wide limit. This number might seem a little arbitrary and if you think that you can review more lines of code - go for it. This is just an average - if the code is really difficult it might not be possible to comprehend more than 200 LOC and if the code is easy you can review 600 LOC. It's always your call. Anyway, find your limit, adjust it a little if needed and stick to it. 

Allow tools to do their work
: {:.margin_bottom} There are tons of linters or spell checkers that can improve your workflow. In [ruby](https://www.ruby-lang.org/) world there is very popular static code analyzer called [rubocop](http://batsov.com/rubocop/). Spend some time configuring it and all code-style related bugs will go away. Reviewers won't spend time pointing out that the alignment is not correct or you misspelled some variable name and they will focus on the logic of the code. 

Use checklists
: {:.margin_bottom} Come up with a list that you go through when you're doing code review. Make sure that this list is available to other members of the team and agreed upon. [This](https://blog.fogcreek.com/increase-defect-detection-with-our-code-review-checklist-example/) is an example of this checklist. It will make your life easier and definitely will help with finding omissions in the code.

Reinforce good practices
: {:.margin_bottom} Code review is not only for pointing out possible issues with the code. It's a communication platform and could be used to give positive feedback as well. When you see something good - comment it. It will help to keep the positive attitude around code reviews as well.


## Consequences

One of the biggest and best effects of regular code reviews is something called "the Ego effect". Everyone starts to pay more attention to the code they submit since no one wants to be known as the guy who always makes silly, junior-level mistakes. It will skyrocket your skills and will boost your understanding of the system much faster and deeper than if you would not do it. Thanks for your attention! Stay tuned!
