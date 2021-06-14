---
title: "Prioritizing Tech Debt"
date: 2021-06-14T16:30:50-04:00
draft: false
tags:
    - product
    - techdebt
---

Tech debt will never get prioritized if you simply complain about it. This is true even if you're persistent in your complaining. You can bring it up repeatedly in the right meetings, with the right stakeholders present, and use the most forceful qualitative language possible to describe the threat this tech debt poses to the organization. Most of the time it still won't get done. By qualitative I mean "we are _very_ far behind on our Postgres version, if we don't upgrade _immediately_ then we're going to get hit with _big_ performance issues in production and we're going to have a _serious_ service degradation event."

Part of the problem is that this can probably said in good faith with just as much urgency about many other tech debt items in the organization. How do the product and engineering decision makers decide to prioritize this tech debt item and not one of the three hundred others in the backlog? In my experience you need to get specific and quantitative, which takes more work but will produce better results.

First off, stakeholders need to know the cost of not taking action. Qualitative words don't communicate cost. You should be able to express cost in numbers. Those numbers don't need to be perfect; they just need to be correct within an order of magnitude to help place this particular tech debt on the scale of severity against other items.

You say it's "really slowing down the dev teams" to work with an old version of Framework X because of such-and-such issues. Well, how would you say that in dollars? To get a dollar amount, how many hours per week does your team waste dealing with this tech debt, on average? You multiply that against the fully-loaded cost of a developer divided by fifty-two forty-hour weeks. You can get general numbers on fully-loaded developer cost from any hiring manager.

You say that it's "going to cause a serious service event" if we stick with the current architecture. Okay, but how many dollars will we lose in renewals? Anybody in sales can get you a number that you can use. Then multiply that times a reasonable number of accounts that might get lost. When you talk to the people in sales, they might even be able to give you specific instances where you lost accounts because of some other service event of similar severity.

You say that it's "going to lead to a security breach" if we stick on an old, vulnerable library version. There are industry numbers out there on the average breach cost that you can use to stick another dollar amount on that risk per year that the tech debt doesn't get addressed.

Once you have a ballpark estimate of the cost of not taking action on this tech debt, you can get into the other question stakeholders will need an answer to: how much time will this take to fix?

To know that, you need more details than "we need to upgrade system X." This will take some conversations with experts in the domain, some planning, and some bookkeeping. Create an Epic or Feature and take an hour to put some cards in there that cover the big parts of what the solution would look like. When you put story points on these cards, estimate high to cover for the fact that you're only spending an hour doing the breakdown. Put extra sandbagging on cards that contain significant unknowns.

Now you have the cost of not taking action and the cost of taking action. This is a quantitative, specific argument for your tech debt that you can bring to the right meetings with the right decision-makers present. You should get better results.