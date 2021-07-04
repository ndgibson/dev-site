---
title: "You're Not Mature Enough For a Pet"
date: 2021-07-03T21:30:56-04:00
draft: false
tags:
  - systems
  - engineering
---

The jargon of "pets" vs. "cattle" may [no longer be used at Google](https://developers.google.com/style/inclusive-documentation#violent-language) but I still hear it thrown around a fair bit in meetings. The term "pet" has an interesting connotation that "stateful systems" - Google's preferred jargon - doesn't convey. The connotation I have in mind is the diligent and tender way you (hopefully) care for your pets. You feed them, give them water, exercise them, play with them, give them your affection, etc. At least, you're supposed to.

When you were a very little kid, your parents might have told you that you weren't mature enough to have a pet. They knew you would probably get bored with that new puppy and stop taking care of it. Then they would get stuck taking care of it and you.

The problem is that in an engineering organization, we're the parents. If we don't take care of our pets, then there's nobody else to pick up the slack.

My current org has dozens of "pet" hosts that require a huge amount of hands-on care to stay healthy. We have Puppet, but that's not a magical band-aid. Basically every engineer from the intern level on up _has_ to have root SSH access in order to get their jobs done. 100 engineers SSH-ing and sudo-ing into dozens of hosts can cause a lot of chaos, with or without Puppet.

This was painfully clear this week when we got hit with a P1 security vulnerability regarding a CVE that came up for our version of [OpenResty](https://openresty.org/en/ann-1019003002.html). My team got the short straw and we had to do a host-by-host manual upgrade of OpenResty. What made it more complicated was that for various historical reasons the hosts were littered with older versions of vanilla non-OpenResty Nginx.

Outdated and vulnerable Nginx code was everywhere:

 - In individual engineers' home directories

 - In service account home directories

 - In every combination of `/etc/` and `/usr/` you can think of

 - Scheduled in `init.d`

 - Scheduled by Systemd

 - Running for who-knows-how-long as manually-initiated processes

 - Installed by `yum`

 - Installed manually

It took so much time to make sure each of the 13 hosts that turned up on the vulnerability report was fully cleaned out and upgraded.

We can take the proper steps and make sure that OpenResty is installed via Puppet so that next time it's just a matter of updating some config and waiting for the next Puppet run, but that won't stop our 100 engineers from SSH-ing in and throwing some new binary of some other tool all over the hosts.

The real problem is that we have pets and we're not mature enough to take care of them. And if you're engineering org is not mature enough to take care of the pets, it's time to either learn or switch to cattle...uh, "stateless cloud systems."



