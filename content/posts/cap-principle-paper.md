---
title: "CAP Principle: Harvest, Yield, and Orthogonal Mechanisms"
date: 2021-09-17T11:20:58-04:00
draft: false
toc: false
images:
tags: 
  - cap
  - architecture
  - systems
---

"Harvest, Yield, and Scalable Tolerant Systems" is an older paper (from way back in 1999) about ways to think outside the CAP principle box when building highly-available systems. You can read it [here](https://radlab.cs.berkeley.edu/people/fox/static/pubs/pdf/c18.pdf).

The CAP principle basically says that when building a system, you can only have two of the following three properties: data Consistency, high Availability, and Partition-resilience. (That last property basically means your system can continue functioning despite a "partition" - that is, an interruption in communication between the parts of your system. For instance, when a node goes down or a host falls over.)

As an example, the CAP principle tells us that we can build a system that is available all the time and always serves perfectly consistent data to everyone who uses it, but it won't be a system that can survive a node falling over. In other words, it won't be 100% available. We can only pick two of the three CAP properties.

The authors (Fox and Brewer) don't disagree with the CAP principle. They agree that, taken strictly, it's impossible to build a 100% partition-resilient system that provides 100% perfect data consistency 100% of the time. They call this the "strong" CAP principle. Think of it as the theoretical/academic/formal version of the CAP principle.

Instead, Fox and Brewer present the idea of a "weak" CAP principle. The reasoning goes like this. We know the strong CAP principle is true, but do real-world scenarios actually require 100% of anything? Are we really doomed to providing 100% of two of the CAP principles and 0% of the third? Or can we find a middle ground where we provide all three CAP principles on a "good enough" basis but fall short of perfection?

Take the second property, high availability. The Internet isn't even up 100% of the time, so a system cannot actually be available 100% of the time. In the best case it's up a certain number of "nines", e.g. 99.999% of the time.

How about data consistency? Does your application actually need to serve perfectly consistent data to all users at all times? Or is there still value in your application even if it serves some users some old data some of the time?

In reality, most applications are still really useful even if they don't have all three properties of CAP. And if you set your sights on real-world "weak" CAP instead of academic "strong" CAP, you could actually build a system that is 99.999% partition-resilient that provides 99.999% data consistency 99.999% of the time. That would be awesome!

The way they recommend going about building such a system is to think about how much consistency, availability, and partition-resilience your system actually needs to provide.

A storefront site doesn't actually need a shopping cart feature to be working in order for people to buy things - they could buy items one at a time. It wouldn't be ideal, but it's better than the whole site having an outage just because the shopping cart component doesn't work.

A social media network doesn't actually need to let the user be able to change settings 100% of the time - users still get value out of their home feed even if the settings component is broken.

What we're describing here is a design where the overall system (the storefront or social media app) is resilient to failure even if some subcomponents (the shopping cart or settings panel) are not.

Fox and Brewer propose two ways to go about designing this kind of system.

The first is to trade "harvest" for "yield". These are terms of art that essentially mean "the fraction of data in the response" and "the probability of completing a request", respectively. The "strong" CAP principle would say that if you can't supply 100% of the data in every response 100% of the time, then you don't have data consistency or high availability. But in reality, most applications still provide value just by providing some fraction of the data to some fraction of the requests. What if you could design a system that reacted to a node crashing by "gracefully degrading", i.e. providing a best-effort fraction of the data to requests?

The authors use the example of a search engine. When a node falls over, it doesn't return errors to all users because it can't provide 100% accurate and fresh search results. Instead, it degrades gracefully and provides mostly-up-to-date search results to all users making requests to the system. If you're using Google, would you be angry that you got search results updated an hour ago instead of a second ago? Would you rather have that stale data instead of a 500 error? Absolutely.

The second strategy for building these systems is decomposing applications using a set of techniques the authors call "orthogonal mechanisms". Another jargony way of saying this that you might be more familiar with is "loosely coupled".

Think about your average web application. A user loads a JS application in his browser and performs some action in the UI. That triggers an HTTP request to the API layer, which queries a database and responds back to the UI. If any layer in this chain of components - the UI, the HTTP layer, the API middleware, the database - has an error, the whole experience fails for the user. The components in this system are "tightly coupled"; they live or die together.

Instead of the entire system breaking if one component breaks, wouldn't it be better if it gracefully degraded but still provided some value to the user?

To do that, you need to loosen up that tight coupling. You need to decompose your application and make these highly-dependent components (the UI, API, database) less dependent on each other. And there are systems that beautifully exemplify loose coupling. Fox and Brewer refer us to SNS, which is based on message streams that use timeouts, retries, and sandboxing.

Fox and Brewer describe a family of mechanisms that  "orthogonal mechanisms", because they help us decompose our applications by making each subcomponent more independent (or "orthogonal") from the others.

The upshot of such a design is that is not only makes your system better from a weak CAP perspective, it also easier makes it easier to maintain and reason about because it forces you to break your system down into smaller pieces.

To review, the CAP principle is a very helpful tool for thinking about tradeoffs in a system: "Here are three good things, you only get two." But in its strong version it obscures the fact that you can shoot for a little less than perfection and get pretty close in all three good things.

To do that, you need to think about 1) what your application really requires and 2) how to break it down into pieces that can fail without taking out the whole system.