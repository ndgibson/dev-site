---
title: "Interviewing: Signal and Weight"
date: 2021-09-10T13:49:30-04:00
draft: false
toc: false
images:
tags: 
  - interviews
---

A candidate you're interviewing picks Java as her language of choice. She's supposed to query a web API and process the response. She spends a lot of time reading through docs and trying to remember how to correctly instantiate the proper sequence of classes that allow her to make an API call, parse the JSON, and manipulate the results. She's typing the whole time and thinking out loud, which is great, but it takes a while to grind through the long sequence of needed structures: `HttpUrlConnection`, `BufferedStream`, `InputStreamReader`, `StringReader`, `JSONObject`, `JSONArray`.

With only a few minutes left, she runs her code and forgets that she hasn't imported the libraries she needs for the structures she's trying to use. And unfortunately some of the third-party imports aren't available in the environment used by the code interview software. Due to these hurdles, she fails to get to a working solution before the end of the alloted time.

Afterward my fellow interviewer and I had a discussion about how much valid signal we could really get from the interview. If the candidate doesn't complete the exercise, or doesn't get to whatever the "meat" of the exercise (data manipulation, for instance) is supposed to be, how can you evaluate that candidate? Maybe we should help more with directing the candidate toward a library that was available inside the environment, or switch to an exercise that is less real-world and more toy-like.

In my opinion, it's all signal. You just have to weight it properly in context.

By signal I mean it's relevant information if someone says they have 20 years of experience in PHP but then can't even manipulate basic structures in PHP during an interview. I think we could all agree on that. In the same way, it's relevant information if someone says they are experienced in Java but doesn't think to check what they can import before getting into a particular implementation.

By "weight it properly", I don't mean we should reject anyone who can't make it through a code interview. I do mean that we need to take that signal in context and think about planning to coach this candidate if we do extend an offer, in order to grow them into thinking more like a senior engineer: run your code often, iterate in short loops, be organized, think through what you're doing.

And if you're more likely to be a candidate during an interview, here are a few pointers.

First, be excellent at the basics of whatever language you choose to do interviews in. If you pick Node.js then you should know the vanilla way to make queries and handle responses, even if you normally let a framework handle that for you.

Second, how you approach the coding exercise is as important as getting it done. Honestly it might be more important. If you're super organized and clear and think through what you're doing, then I'll be understanding if you don't finish the exercise. If you hustle and slop things together and flail around, I'll be shocked if you _do_ finish. Because usually these candidates don't. Slow and steady wins the race, etc.