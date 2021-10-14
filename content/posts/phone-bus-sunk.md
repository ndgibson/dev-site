---
title: "Phone a Friend, Sunk Costs, Bus Insurance"
date: 2021-10-14T19:56:25-04:00
draft: false
toc: false
images:
tags: 
  - lessons
---

TL;DR:

* If you have a gut feeling that a card isn't going to go well when you pull it, don't lie to yourself - get help early.
* Beware the sunk cost fallacy - even if you've spent N hours on a certain approach, throwing away those hours to try a different approach can be better than investing N+1, N+2, etc. in the same approach that isn't working.
* Do your work in the open and document as you go, so that other people can pick up your work and meet team commitments even if you get "hit by a bus".

In the second half of our sprint, the engineer working a key bugfix card got sick and ended up being out on PTO the whole week. I stepped in to pick up the card on Tuesday, after spending Monday wrapping up what I already had in progress. I had a gut feeling that the card wouldn't go well. It was a bug in a complex system with a poor local development story and stale documentation. I hadn't been involved in the work because I had been on production support the week before, and most of the people who knew the system had left the organization. The engineer who had pulled the card had a WIP branch but no notes.

I should have called in some help right at the beginning. All my gut-feeling worries ended up coming true: it was hard to get a local dev environment working, it wasn't clear where the WIP was in terms of more or less done, the test pipeline didn't work, etc.

And as I started working on it, I kept on delaying the summoning of cavalry because I thought that with just another hour of digging I could get unstuck. This is sort of like the sunk cost fallacy - my current approach (working on it solo) isn't working, but I've spent so long on it, I just need another hour after my next set of meetings and then I'll figure it out... In reality, I should have just bailed on the solo approach and knocked on virtual office doors until I found somebody who knew more than I did about the system and could get my untangled.

And lastly, it all would have been easier if our organization had a better habit of working out in the open and documenting things. This isn't a criticism of the first engineer - after all, the entire system is undocumented and obscure! This has been a problem since before either of us joined up. But in the future we can start to mitigate this problem in a couple ways: doing our work in the open with Draft PRs, writing up notes in shared documentation tools, and documenting code like it'll be used by customers (future us).