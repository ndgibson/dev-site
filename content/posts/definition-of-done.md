---
title: "Definition of Done: Why and What"
date: 2021-07-09T09:27:04-04:00
draft: false
toc: false
images:
tags: 
  - architecture
  - systems
---

To a large degree, engineering is a matter of navigating tradeoffs. Whenever you build a system, you have a range of options that trade off time/cost investment for quality. It’s a spectrum that ranges from a barely working system to a perfect system. At the low end of the spectrum you spend a day and whip up something that works - but just barely and isn’t sustainable. At the high end you have the perfect system that takes an infinite amount of time to build.

The [Project Management Triangle](https://en.wikipedia.org/wiki/Project_management_triangle) has been around for almost a century. Budget, scope, and time are corners of a triangle. It has its flaws, but the idea is that these three things are tradeoffs. If you want to shorten the time to deliver a project then you have to decrease scope and increase budget. If you want to increase scope you have to increase budget and time.

The problem with the PMT is that often stakeholders come to engineering and say that budget, scope, and time are all fixed. "Now build this system." And often times engineering will do that by unofficially reducing quality. We're eager to comply and shift our system to the left on that spectrum of low-to-high quality. We ship a product without monitoring, or CI/CD, or testing.

The point of the Definition of Done (DOD) is to have an official minimum for Quality, the invisible "fourth side" of the PMT. The DOD is a list of functional and non-functional requirements that a system must have in order to be considered Done. Quality is still flexible to a degree: you could build in more quality than the Definition of Done requires, for example. But you cannot lower quality below what the Definition of Done specifies as the minimum.

What are the benefits of having a DOD and sticking to it? In the short term, a DOD makes the conversation about the PMT tradeoffs of budget, scope, and time more meaningful because we no longer can cheat on quality. We have to make painful decisions and have honest conversations with Product about what we can get done. In the medium term, a DOD moves your systems to the right on the spectrum, toward higher reliability and better sustainability. In the long term, a DOD will reduce the time it takes to deliver projects because you'll have less tech debt dragging down your velocity. Also in the long term, a DOD will make production support less painful.

The DOD should apply to everything you build: services, applications, pipelines, models. Everything in the DOD should be achievable now given your existing patterns and technology. It's not exhaustive, though. They say that no law code can address every potential situation that may come up in a society; that’s why you need judges and courts and jurisprudence. In the same way, some wisdom will be needed to apply your DOD to each individual project. This process of being engineering judges and using wisdom takes several forms:

- Referring to the DOD when planning epics.
  
- Referring to the DOD when conducting code reviews.
  
- Referring to the DOD when having formal design reviews for big features that require a technical design doc.

Your DOD should cover the different service levels of your Software Development Lifecycle. Here are a few potential levels:

- Prototype: Move quickly. Make high-value, minimally-burdensome investments for the future in case the prototype gets turned into a product.

- Alpha: Build quality in from the beginning so that the General Availability release is more of a customer-facing decision and less of a heavy engineering lift getting the system production-ready.

- General Availability: Communicate to customers, hook up production support, finish up documentation. Drink some champagne.

You may also want an aspirational section in your DOD for requirements that ideally should be in the DOD but you lack the patterns or technology to implement. This is your Lodestar section.

Your DOD should also cover the various engineering pillars at each of the different service levels. Here are a few major ones:

- Design: How do you design a new system? What are the documents and processes to make sure the design is good?

- Develop: What is your development loop for your systems? What tools and standards do you use?

- Build: How will your systems be continuously integrated, tested, and deployed?

- Test: How will you test your code?

- Operate: How will you instrument your systems with monitoring and metrics so that they can be supported in production?

Your DOD shouldn't be top-down or written in stone. It should be a dynamic document that you write in collaboration with the whole engineering group. And even once you have a DOD, you will all have to work together to help keep each other honest that when you say a card or an epic is "done" it really _is_ "done" according to the Definition of Done.



