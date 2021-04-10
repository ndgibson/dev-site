---
title: "Too Expensive to Build Right"
date: 2021-04-09T20:45:24-04:00
draft: false
toc: false
images:
tags: 
  - applications
  - architecture
---

I've worked at companies where engineering didn't know how to build maintainable systems. We had the tools but not the knowledge. Also, thankfully, I've worked at companies that had the knowledge and the tools. It was still hard to execute, because nothing in real life is easy and there are always competing urgencies.

Now I'm dealing with another kind of challenge. This time it's a company that knows how to engineer well at a theoretical level, but the overhead of trying to execute on a plan for a sustainable system is so high that nobody does it. Case in point: any card dealing with infrastructure has a huge tax on it. It can take days to refresh certs on a pet host. So, once you present your plan for building a highly available, monitored, automated, continuously integrated and deployed application, it costs too much to fit on the roadmap because of all that infrastructure tax.

None of the tricks that smaller companies use to reduce this overhead apply, because nothing can face the public internet. So we can't outsource our production crash monitoring to [Sentry.io](https://sentry.io) or offload the burden of metrics collecting to a platform with incredible APIs and libraries like [Datadog](https://www.datadoghq.com/). Or technically we can, if we take the time to deploy the hosted version on our own infrastructure and are willing to take on the responsibilities to support it.

This is all a tough sell to Product, which is under pressure to deliver important things to our internal customers.

What to do? We can try to make some big advances by planning epics that fix pain points (say, host our own Sentry instance) and advocating to Product that these epics will help customers as well as engineering. I've done that before with some success. But I suspect that the real way we will make progress is slowly, one card at a time, one sprint at a time. We need to look for opportunities to put an extra point on a card and expand its scope to cover reducing the cost of building systems while also providing value to the business.

For instance, don't just make a 2-point card to write a script for end-to-end browser testing that runs on a dev laptop. Make it a 3-pointer and have some acceptance criteria in there about making progress on running the script from the Jenkins host. Etcetera.

As with many things, we'll see how this goes.

