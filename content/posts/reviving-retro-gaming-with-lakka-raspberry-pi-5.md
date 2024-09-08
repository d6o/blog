+++
title = 'Reviving Retro Gaming With Lakka Raspberry Pi 5'
date = 2024-09-08T18:09:18-03:00
draft = false
tags = ["Retro Gaming", "Raspberry Pi", "Lakka OS", "Game Emulation", "RetroArch", "Sega Master System", "Raspberry Pi 5", "Golang", "Atari", "Sonic the Hedgehog", "DIY Gaming Console", "Gaming Nostalgia", "Tech Projects"]
+++

**Reviving My Mom's Love for Retro Gaming with Lakka and a Raspberry Pi 5**

A few months ago, I visited my parents, which coincided with Apple's recent shift in allowing emulation apps on their
devices. RetroArch had become available on the Apple TV, and that was the catalyst for this entire journey. Growing up,
I had an Atari (though I'm not sure which model), a Sega Master System Mark III, and later a PlayStation 1. What stuck
with me most was how much my mom loved the Sega Master System, particularly *Sonic the Hedgehog*. She was the only one
who could beat the first Sonic game—something I never managed to do as a kid and still haven't to this day.

During that visit, I took full advantage of RetroArch on the Apple TV, setting it up with some games and pairing two
DualSense controllers. Everything worked like a charm—at least on two of their Apple TVs. My mom, who hadn't played in
over two decades, could now enjoy Sonic the Hedgehog anytime she wanted. It was a nostalgic thrill for both of us.

A few days later, my mom called to say that the games had disappeared. I was puzzled. RetroArch was still installed, but
all the games were gone. After some digging, I discovered that Apple TV doesn't guarantee app storage; it deletes files
it deems unnecessary, including my mom's beloved game files. With no way to force the files to stay, I knew I needed a
more permanent solution.

### Enter the Raspberry Pi 5 and Lakka

Back at my parents' house this time, I came prepared. I ordered a Raspberry Pi 5 and started looking for the best way to
set it up for emulation. RetroArch had worked well previously, so I started there and soon discovered *Lakka*, an OS
specifically designed for game emulation. It seemed perfect for my needs.

Lakka is a lightweight Linux distribution that turns your Raspberry Pi into a full-blown retro gaming console. Built on
RetroArch, it supports a vast number of systems and is incredibly user-friendly. Installation is simple: just flash the
Lakka image onto an SD card and upload some ROMs.

I flashed Lakka onto a 64GB microSD card, installed it on the Raspberry Pi, and started downloading ROMs
from [Myrient](https://myrient.erista.me/files/), a trusted source for game files. I even wrote a quick Golang program
to automate downloading and extracting ROMs, then enabled SSH on the Raspberry Pi to start uploading the games.

### The Storage Problem

Things were going smoothly until I hit an unexpected issue: "no space left on device." I was only uploading Atari
ROMs—tiny files of about 3-4 KB each—so I was confused. After SSHing into the Raspberry Pi, I found that the `/storage`
partition had only a few megabytes available, even though I was using a 64GB microSD card.

The [Lakka installation guide](https://www.lakka.tv/get/macos/rpi/install/first-boot/) mentioned that the system should
expand the file system on the first boot, but it seemed like this hadn't happened. After searching Reddit and
StackOverflow, I found some old posts on how to manually resize the partition, but they were outdated and unclear. Then,
I turned to the LibRetro Discord channel for help, and within a few minutes, I had an answer. A user named `jdalmanza`
pointed me to a recent message that explained how the latest Lakka release had an issue with SD cards larger than 32GB.
The good news: the issue was fixed in the latest nightly release.

I downloaded the nightly release but also asked if there was a way to manually resize the partition. `jdalmanza`
provided a helpful solution: run the `resize2fs` command. For me, that meant:

Finding the device path:

```
Lakka:~ # df -h
Filesystem                Size      Used Available Use% Mounted on
/dev/mmcblk0p2           26.0M      4.8M     20.5M  19% /storage
...
```

Resizing the filesystem:

```
resize2fs /dev/mmcblk0p2
```

This worked like magic! My storage went from 26MB to 55.3GB in seconds:

```
Lakka:~ # df -h
Filesystem                Size      Used Available Use% Mounted on
/dev/mmcblk0p2           55.3G      4.8M     55.3G   0% /storage
...
```

### Filling the Pi with Games

With the storage issue resolved, I improved my Golang program to download all the ROMs for each system my mom loved:
Atari 2600, Sega Master System, and others. In total, I downloaded and uploaded almost 20,000 games via SSH over Wi-Fi,
which took some time, but it worked flawlessly.

Here's a snapshot of the game systems I set up:

```
Atari - 2600                 Atari - ST                Nintendo - Game Boy Advance  Sega - Game Gear
Atari - 5200                 Commodore - Amiga          Nintendo - Game Boy Color    Sega - Master System - Mark III
Atari - 7800                 Microsoft - MSX            Nintendo - NES (Headered)    Sega - Mega Drive - Genesis
Atari - 8-bit Family         Nintendo - Game Boy        Nintendo - SNES
```

After uploading all the ROMs, I indexed the games in Lakka, which took about 15 minutes. The result? A powerful retro
gaming machine with all the games my mom used to love, ready to be played any time she wants.

### Conclusion

This project was a labor of love, combining nostalgia and tech to create something special for my mom. From overcoming
storage issues to automating ROM downloads, it was a rewarding experience. Now, thanks to Lakka and a Raspberry Pi 5, my
mom can revisit the glory days of *Sonic the Hedgehog* and much more.

The best part? Seeing her joy when she picked up the controller again, ready to relive those cherished moments.

If you're looking to bring retro gaming back into your life or want to share those memories with loved ones, I highly
recommend trying Lakka on a Raspberry Pi. It's a fun, rewarding project, and maybe you'll even beat *Sonic*
this time!
