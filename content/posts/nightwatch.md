---
title: "NightwatchJS on Dockerized Chrome"
date: 2021-04-15T16:11:24-04:00
draft: false
toc: false
images:
tags: 
  - javascript
  - testing
---

I spent some time this week setting up a webapp repo with [NightwatchJS](https://nightwatchjs.org/) for UI automation testing. It's not a framework I have experience with (I'm more used to Pytest) but we had some existing Nightwatch tests in the organization so I decided to roll with it.

I like the idea of writing UI automation testing in JS instead of Python. Typically it's the frontend engineers who write the UI automation anyway, and inevitably pieces of the frontend implementation (e.g. CSS selectors) end up in the automation tests. Having those tests in the same language and repo as the frontend app seems intuitive.

Here was my success criteria:
- have the Nightwatch tests in the same repo as the application
- be able to run the tests on a local machine with minimal tooling
- be able to run them in a docker container for CI/CD pipelines
- have the tests run against the actual application in staging, not a local version

It was pretty straightforward, with the usual obnoxious hiccups around getting Chrome to cooperate. There's lots of outdated information out there about how you need to install Selenium, Xvfb, Chromedriver binaries, etc. That might have been true with older versions of Nightwatch but it's pretty clean now. You can use Chromedriver as an NPM module and keep the tooling dependencies to a minimum.

My `package.json`:

```json
{
  "devDependencies": {
    "chromedriver": "^89.0.0",
    "nightwatch": "^1.6.2"
  }
}
```

That's it. The critical pieces (and the gotchas) come in your `nightwatch.conf.js` file.

```js
module.exports = {
  webdriver: {
    start_process: true,
    server_path: "node_modules/.bin/chromedriver",
    port: 9515
  },
  test_settings: {
    desiredCapabilities: {
      browserName: "chrome",
      chromeOptions: {
        args: ["--headless", "--no-sandbox", "--disable-gpu"]
      },
      alwaysMatch: {
        acceptInsecureCerts: true
      }
    }
  }
}

```

You may not need `acceptInsecureCerts` but in my case I needed to workaround not being able to resolve SSL certs for an internal application.

One of the test cases I wrote needed to validate that a CSV file could be downloaded from our React app. Normally in Chrome you get a download confirmation window if you click a download link, which is a pain to deal with in a test. Here's how to update your `nightwatch.conf.js` file to get downloads to be automatic and land in the directory of your choosing:

```js
{
  chromeOptions: {
    prefs: {
      download: {
        default_directory: require("path").resolve(__dirname + "/tmp"),
        prompt_for_download: false,
      },
      profile: {
        default_content_setting_values: {
          automatic_downloads: 1,
        }
      },
    }
  }
}
```

As for the Dockerized approach, my `Dockerfile` is `FROM` an internal Centos7 image.

```
# install chrome
RUN yum install -y wget && \
  wget https://dl.google.com/linux/direct/google-chrome-stable_current_x86_64.rpm

# set up the project
RUN mkdir /automation
COPY . /automation
WORKDIR /automation
RUN npm install

# start the nightwatch tests
CMD npx nightwatch
```

Since we're using the `CMD` directive, all we have to do is build the image and then run it and the tests will begin immediately.

```bash
docker build . -t automation:local
docker run automation:local
```

As for actually working with Nightwatch...I like it. The API is pretty well documented and it has all the features you need for basic browser automation. And as you can see, setup was a breeze. There's two aspects of the API that I think could be improved, though.

First is the assertion APIs. Nightwatch offers `assert`, `verify`, and `expect`, each of which work differently. But all three variants are tied to the context of a `WebElement`: `assert/verify` take a selector as the first argument, and `expect` is supposed to be chained with the `element` finder API.

What if I just want to fail the test if some variable does not equal a value? Do I really have to add another dependency like [Chai](https://www.chaijs.com/) or be a barbarian and use an `if` with a `throw new Error`?

The other thing is that the docs encourage you to use `PageObject`, but that API is not well-integrated with Nightwatch's general `browser` API. I really like the `PageObject` abstraction layer. It keeps your test logic concise, and gathers potentially brittle element selectors in one module. 

The problem is that there are critical Nightwatch features (like window management, pausing, grabbing all elements that match a selector) that are only available from the `browser` API. Nightwatch wants you to chain together calls but there's no graceful way to swap back and forth between `browser` APIs like `pause` and your `PageObject` custom commands.

It's possible there are idiomatic solutions to these initial problems that I'll discover as I spend more time with Nightwatch. But so far, I'm impressed.