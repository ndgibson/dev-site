---
title: "Debugging Takeaways: Console Logging and Dependencies"
date: 2021-05-21T16:23:37-04:00
draft: false
tags:
    - debugging
    - python
    - flask
---

TL;DR:

- don't forget to use console logging to figure out problems

- pin your dependencies or your code isn't really in source control

- invest in automated builds and deploys

I spent Thursday frustrated with a Python application. I had made some seemingly innocuous changes related to fetching DB connection secrets and deployed the new code to our staging environment, only to find that Gunicorn failed to start due to an error from Flask:

```
Error: Working outside of request context.

This typically means that you attempted to use functionality that needed an active HTTP request. Consult the documentation on testing for information about how to avoid this problem.
```

Flask wasn't even booting up. I couldn't see anything about my changes that could possibly have impacted something as basic as Flask's initial setup and execution. Even worse, I couldn't replicate the failure on my local machine. It only failed inside Docker on the staging environment. (This was an important clue that I should have noticed sooner.)

I wasted time for quite a bit making sure that my change to hit a new service for DB connection secrets wasn't causing the problem. I also tried to set up debugging on a breakpoint inside the staging app, but getting debugging working through Gunicorn to Flask is non-trivial and I couldn't figure it out.

The next day I had a fresh start and also some help from a coworker who was game to lend a hand. He quickly made a helpful back-to-basics suggestion that I had forgotten on Thursday:

> "Since we can't get an interactive breakpoint, let's just add a bunch of console logs."

After about thirty minutes of adding logging statements and binary-searching the problem down to a particular line, two things became clear:

- the error was originating in a dependency
- nothing in my diff had any correlation to the error

The error was being thrown from a [flask_injector](https://github.com/alecthomas/flask_injector) call to inject per-request dependency bindings into our app. The flask_injector module didn't like something about iterating over a dictionary of [Jinja2](https://pypi.org/project/Jinja2/) globals, one of which was a `LocalProxy` instance.

That was when my coworker had another great idea:

> "Let's check the dependency versions on your local machine versus the Docker image."

Using the python REPL we quickly checked the versions of `flask_injector` and `jinja2`:

```
> python3
python3$ import flask_injector
python3$ flask_injector.__version__
python3$ import jinja2
python3$ jinja2.__version__
```

By doing this on my local as well as the broken staging environment, we found that we had different versions of Jinja2. Additionally, Jinja2 was not pinned in the `requirements.txt` file! So we pinned it to the working version from my local machine, thinking it would fix the problem.

It didn't. We got the same error as before.

Then we found this [telling commit](https://github.com/alecthomas/flask_injector/commit/b57e5e054a8e3886f8804bdd367a8b353c4b3b19) to our flask_injector module. A quick check of the `Werkzeug` module revealed that it, too, was different between local and the staging environment.

Pinning it to my working local version fixed the problem.

Lessons learned?

- Don't be afraid to go back to basics and use console logging to figure out a problem, even in third-party libraries. You don't _need_ to be able to set breakpoints to figure out a problem. This is a really easy thing for frontend-oriented developers to forget, because we always have awesome browser devtools for easy breakpoints and interactive debugging. We forget console logging is even an option.

- You need to pin dependencies. This is a really great experience in Ruby with Gems, and Python has modules like [pip-tools](https://github.com/jazzband/pip-tools) that are very similar. If you aren't pinning dependencies, then you don't have deterministic builds even if you're Dockerized. Think about it: a big portion of your app is in the dependencies. Maybe 90% on a small app. If those dependencies aren't described in some format like `requirements.in` that can be checked into source control, then 90% of the code needed to deliver business value _isn't checked into source control_.

- Invest in automated builds and deploys. This particular app is still a manual build on local machines. We wasted a lot of time manually running the build and deploy process to test changes on staging and we didn't history of previous build logs (e.g. Jenkins console output). With build logs, it would have been trivial to diff the versions of `Werkzeug` being installed on staging vs. production and note that the version had drifted.