---
title: "FM Microbroadcasting"
date: 2021-11-12T17:57:36-05:00
draft: false
toc: false
images:
tags: 
  - radio
  - raspberrypi
---

I grew up listening to a lot of radio as well as recordings of audio dramas of various kinds. (Think things like the epic BBC Radio 4 Lord of the Rings [adaptation](https://en.wikipedia.org/wiki/The_Lord_of_the_Rings_(1981_radio_series)) from the 1980s.)

Recently I decided to use my [Raspberry Pi 4](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/) for something more than just blocking ads and trackers with [Pi-hole](https://pi-hole.net/). I've started broadcasting my own little [Part 15](https://en.wikipedia.org/wiki/Title_47_CFR_Part_15) radio "station".

Why might you do this instead of more modern tech? Well, FM has better signal strength and propagation than your average Bluetooth or Wifi setup. I have this FM Transmitter in my basement in an under-stairs closet and I can get the signal on a cheap radio wherever I go on my property. I have a pretty good router in the same closet and the signal dies in the carport. Don't get me started on Bluetooth! Sometimes it can't even propagate through the entire upstairs.

And with FM there's no pairing or passwords or devices that need charged nightly. You just take a radio, pop in a few batteries that will last for months or years, and turn it on.

Here's the hardware/software list:

- [Raspberry Pi 4](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/)
- [C. Crane FM Transmitter 2](https://ccrane.com/digital-fm-transmitter-2-for-sending-near-broadcast-quality/)
- [VLC-Scheduler](https://github.com/EugeneDae/VLC-Scheduler)
- [VLC](https://www.videolan.org/vlc/)

## 1. Setup the Raspberry Pi

I won't go through every detail of the standard setup process for a Raspberry Pi. Off the top of my head you need:

- Obviously some Linux distro, e.g. Raspbian
- VLC installed and on your `PATH`
- Python 3.6+
- The VLC-Scheduler source checked out locally on your Pi's storage

Make sure that you turn up the volume on your Pi's headphone jack. You can use `alsamixer` or other tools for this. Do the same for the Master volume.

## 2. Setup VLC-Scheduler

You can build a binary for VLC-Scheduler but I've just been running the Python script directly. So check out the VLC-Scheduler source from Git, create a `vlcscheduler.yaml` file configured to play streams or audio from local storage, and boot up the program with Python!

I made a couple tweaks to get it working the way I wanted. (If I keep using it, I'll fork the repo.) First I added a line [here](https://github.com/EugeneDae/VLC-Scheduler/blob/master/src/vlc.py#L56) to pass the `--volume-step` [option](https://wiki.videolan.org/Documentation:Command_line/#Other_Options) with a value of `256`:

```
command = [
    self.config['path'],
    '--volume-step', '256',
    '--extraintf', self.config['extraintf'],
    '--http-host', self.config['host'],
    '--http-port', str(self.config['port']),
    '--http-password', str(self.config['password']),
    '--repeat', '--image-duration', '-1'
] + self.config['options']
```

I also found a weird bug where VLC-Scheduler wasn't able to detect the length of MP3 files and just played the default 60 seconds. I fixed it by adding a longer pause to the code that checks the file length from VLC. You can read the issue someone else filed [here](https://github.com/EugeneDae/VLC-Scheduler/issues/2) and see the code in question [here](https://github.com/EugeneDae/VLC-Scheduler/blob/master/src/vlcscheduler.py#L69):

```
if play_duration < 0:
    await asyncio.sleep(1)
    actual_duration = player.status().get('length', 0)
    if actual_duration <= 0:
        actual_duration = config.IMAGE_PLAY_DURATION
    play_duration = min(actual_duration, -play_duration)
```

That was about all I tweaked with VLC-Scheduler. Eventually we can get VLC-Scheduler scheduled to run when the Pi boots; right now you can just SSH into the Pi and run it as a background process with `tmux` until you get everything figured out:

```
> tmux
> python3 src/vlcscheduler.py
> [CTRL+B][D]
> tmux ls
> tmux attach -t <session> 
```

etc.

## 3. FM Broadcasting

The first thing you want to do is go to [radio-locator.com](https://radio-locator.com/cgi-bin/vacant) and use their tool to find a (relatively) vacant FM channel near you. I'd recommend letting the site detect your location, because putting in your ZIP code or your City will be less accurate. For instance, in my location it recommended 103.3 FM when I put in a City/ZIP and 90.3 FM when I allowed for exact location detection. The 90.3 FM station worked much better.

Once you have your station, set it on your C. Crane FM Transmitter 2.

I certainly wouldn't recommend peeling off the sticker from the back and using a Phillips screwdriver to adjust the hidden potentiometer for a stronger signal. You can get in trouble with the FCC and void your warranty doing this. You definitely shouldn't use this hidden feature to dial in exactly the signal strength you need to cover your house and/or yard. I also wouldn't know that for some reason the instructions online say to turn it clockwise but on my unit the signal got stronger when turned counter-clockwise.

## 4. Tune In and Enjoy!

Using this I'm able to get a clear and loud FM broadcast anywhere on my property, even from little Walkman-style radios like the Sangean DT-800, Sangean HD-14, and the C. Crane Pocket. My bigger portables like the Sangean HD-16 and Sangean PR-D4W sound incredible. The C. Crane transmitter punches through drywall, wood framing, brick walls, whatever. It's really an excellent little performer for the price and power requirement.

VLC-Scheduler is no longer maintained, which is too bad, because it's some neat software. You can set up your own radio station complete with different programming blocks, interstitial station identification and "ads", whatever you want. It's up to you.

The major downsides are that it doesn't remember where it left off when sequentially playing through a folder of MP3s. I'd really like that feature so I could play one episode of a serial radio drama per day at the same time, in sequence from beginning to end. Again, if I stick with this for my microbroadcasting then I'll consider opening a PR or forking the repo.

## 5. Crontab & Daily Schedules

I won't go into a ton of detail on how to get VLC-Scheduler running as a cron, but I will share some specific notes for using cron to run VLC-Scheduler with different daily schedules. Maybe you want one schedule for Mon-Fri, but a different schedule for the weekend.

You can pass a `VLCSCHEDULER_YAML` environment variable (doc [here](https://github.com/EugeneDae/VLC-Scheduler#vlcscheduleryaml)) to specify different schedules for different days.

And you can use `timeout` to kill your VLC-Scheduler service at the end of your "broadcast day" so that you don't end up with multiple VLC-Schedulers running at one time. For instance, if you were going to start broadcasting at 6AM and conclude at 9PM you could prefix your crontab command with `timeout 15h`.

Just remember that in crontab you don't have the same `PATH` variables as from your normal shell. You'll probably need to end up prefixing each of the components of your command (`python`, your YAML file) with the full path.

If you see strange errors from VLC in the crontab runs (install `postfix` and configure it for local storage if you can't see the output of your crons) that don't appear in the normal shell, you probably need to specify the headless CLI version of VLC in your YAML file:

```
vlc:
    path: '/usr/bin/cvlc'
```

So your crontab might be:

```
0 6 * * Mon-Fri VLCSCHEDULER_YAML=/path/to/vlcscheduler-workday.yaml timeout 14h python3 /path/to/src/vlcscheduler.py
0 6 * * Sat-Sun VLCSCHEDULER_YAML=/path/to/vlcscheduler-weekend.yaml timeout 14h python3 /path/to/src/vlcscheduler.py
```

## 6. Relaying Streams

It's not really obvious from the VLC-Scheduler documentation, but the tool also supports playing internet audio streams. You could use this to mirror your favorite internet radio stream onto your FM microstation. Here's the [example XSPF file](https://github.com/EugeneDae/VLC-Scheduler/blob/master/docs/dam-square.xspf).

Just replace the content of the `<location>` tag with your stream, and then put the XSPF file into a directory and configure it like any other local audio source:

```
- path: /path/to/stream-directory
```

Basically, if VLC can open the stream, then you should be able to schedule it using this approach. I haven't tried it yet, but I'm guessing that installing a [YouTube playlist parser](https://addons.videolan.org/p/1154080) for VLC would allow you to use the same XSPF format for streaming a YouTube playlist to your microstation.