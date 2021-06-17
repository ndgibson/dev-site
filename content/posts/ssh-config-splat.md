---
title: "SSH Config Wildcards"
date: 2021-06-17T19:25:14-04:00
draft: false
tags:
  - ssh
---

I really shot myself in the foot this week. Or, rather, my past self (six months ago, when I was onboarding) shot my current self in the foot. I can't even remember why I did this - it was either to connect to some set of hosts or to fix an SSH workflow in Git - but this was my `~/.ssh/config` file:

```bash
Host *
  AddKeysToAgent yes
  IdentityFile ~/.ssh/id_ed25519
```

What this means is that for _every_ SSH connection (and SFTP connection, more on that later) I make from my machine, use the public key located at `~/.ssh/id_ed25519`. Like I said, I don't remember, but I bet I did this at the end of the day when I was frustrated with some problem and just _wanted to get unblocked so I could do some real work_.

Clearly for the vast majority of SSH connections I've done in my six months on the job, this isn't a problem. I completely forgot I'd even set this up, or why. Until this week.

I was trying to connect via SFTP using [Cyberduck](https://cyberduck.io/) to an IT server that acts as a midway transfer for data coming out of a highly secure system. A different team takes data off an ultra-secure host and puts it in this midway FTP server, and then I needed to connect to the FTP server to get that data off.

Well, it didn't work. The connection parameters all looked right. The error message coming back from the server was completely useless. `traceroute` showed that I could connect to the FTP server from my machine through the proper routing path. A teammate also couldn't connect, for the same reason. (I guess he had the same bad SSH config... I need to follow up on that, actually.)

Support was completely baffled because they could connect and I could not, but everything they checked on my machine looked fine: VPN, various auth tokens, OSX version, Cyberduck version, on and on and on.

Eventually Support started sipping from the firehose of logs coming off this FTP server while I repeatedly tried to connect. We found the error: "unsupported public key algorithm." But that made no sense! In Cyberduck I was explicitly not sending an SSH key!

Except I was. When we finally checked the SSH config, we found the culprit in the `Host *` line. After removing that line and then rebooting, I got in.

So, lessons learned:
- When you're trying to fix tooling, especially in a domain you don't know very well, it's fine to try sweeping changes. Just don't walk off and leave it like that when you sign off at the end of the day. Take the time to figure out what's really going on and make sure that your final changes are 1) intelligent and 2) address the problem correctly. If you just leave your crazy band-aids and monkey-patches in place and go off and forget about them, you'll suffer for it later. 
- Keep track of your system-wide changes:
  - SSH config
  - bash_profile, bashrc, zsh profile
  - PATH variables
  - etc...
- Be careful when using splats in SSH config.