---
title: "Basics of node-sass"
date: 2021-10-08T16:58:26-04:00
draft: false
toc: false
images:
tags: 
  - node
  - node-sass
  - javascript
---

SASS stands for *S*yntactically *A*wesome *S*tyle*S*heets. It's a scripting language that looks like CSS but isn't, and gets interpreted/compiled into CSS. The [original SASS compiler](https://sass-lang.com/ruby-sass) was written in Ruby. Ruby is fun to write, but it's not super fast and not everyone wants to have Ruby in their toolchain. (Ever corrupted your Ruby environment on OSX? It's a nightmare.)

So later on Michael Mifsud did a C++ port of the Ruby Sass compiler called [LibSass](https://github.com/sass/libsass), which as you may expect offered much better performance and removed the Ruby requirement from the SASS compiling process as well.

However, LibSass is "just" a C library with an API. It requires language-specific wrappers (or implementors) to actually be useful. For JavaScript projects, one implementor is [node-sass](https://github.com/sass/node-sass). It's since been deprecated in favor of [Dart Sass](https://sass-lang.com/dart-sass), which I hope to look at soon. But if you've done much JavaScript development, you've surely come across node-sass.

So, node-sass is a Node.js implementor for the C API offered by LibSass. Where node-sass differs from most other Node modules you'll install is in its platform-specific nature. Because LibSass is a C-family library, you need to have a LibSass (and therefore a node-sass) binary that has been built for your operating system and processor architecture, just like other C programs. Enter the node-sass `.node` file. A `.node` file is a binary file used for [Node add-ons](https://nodejs.org/api/addons.html). You've mostly likely seen [one of these](https://github.com/sass/node-sass-binaries) in a project you've worked on.

In short, node-sass uses a `.node` binary to wrap the LibSass compiler so you can compile SASS files to CSS from your Node project. (I say Node project, but you'll do this for non-Node JavaScript applications like React as well. The key is that Node.js is part of your build toolchain even if the final product is a React bundle asset.)

One of the most common errors you'll see as a newbie - especially if Docker is involved in your toolchain and you're swapping between an OSX host platform and a Linux docker image - is a complaint from node-sass that your binary was compiled for a different platform or architecture. The internet will tell you to just run `npm rebuild node-sass` but usually that's not needed. What you can do is just go get the correct binary from [the list](https://github.com/sass/node-sass-binaries) and put it into your source control. Then, before you `npm install`, specify where your binary is located with an environment variable:

```
ENV SASS_BINARY_PATH=/app/dependency/linux-x64-72_binding.node
```

When you `npm install`, node-sass will run an `install.js` script that will either go out to the internet to get a binary or use a local one. The `SASS_BINARY_PATH` environment variable will tell it to use the one you already downloaded. No need to rebuild or have access to the public internet from your CI/CD pipeline. You can even have multiple binaries and set `SASS_BINARY_PATH` for OSX or Linux as needed.