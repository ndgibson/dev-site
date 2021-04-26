---
title: "Terminal WCPE Using VLC"
date: 2021-04-26T09:35:32-04:00
draft: false
toc: false
images:
tags: 
  - untagged
---

One of the pleasures of living in North Carolina is that we get [WCPE](https://theclassicalstation.org/) on the radio. 24/7 classical with informed presenters and no ads. Of course you don't have to live in North Carolina to enjoy it; they stream online. Here's a quick and easy way to play it from your terminal during the workday:

```bash
$ brew install vlc
$ alias wcpe="vlc --quiet --intf dummy http://playerservices.streamtheworld.com/api/livestream-redirect/WCPE_FM.mp3"
```

The command line flags here tell [VLC](https://www.videolan.org/vlc/) to launch with logging suppressed (`--quiet`) and to use a dummy interface (`--intf dummy`). This way you get a headless, terminal-only experience. Simple!

This is assuming you're on OSX with `brew`, but from Windows using `chocolatey` it's about as easy to get VLC installed and accessible from the terminal:

```
> choco install vlc
```

Or on Linux:

```
% sudo apt install vlc
```