---
layout: post
title: "Fix cracking sound when using headphones on Arch Linux"
date: 2019-01-12 20:57
comments: true
---

After fiddling with this for a while the source of my issue got resolved by adding custom parameters to my audio kernel module.

You can find out the module name by doing:

```bash
lspci -nnk | grep -iA2 audio
```

...which in my case, a Lenovo Thinkpad X1 Carbon 6th gen, returned:

```bash
00:1f.3 Audio device [0403]: Intel Corporation Sunrise Point-LP HD Audio [8086:9d71] (rev 21)
Subsystem: Lenovo Sunrise Point-LP HD Audio [17aa:225c]
Kernel driver in use: snd_hda_intel
Kernel modules: snd_hda_intel, snd_soc_skl
```

What interests us here is that module named `snd_hda_intel`. The fix requires you to pass a `model` parameter to it —
you can find a [list of models here](http://git.alsa-project.org/?p=alsa-kernel.git;a=blob;f=Documentation/sound/alsa/HD-Audio-Models.txt;hb=HEAD).

There are a couple of ways to play with these values until you find the right one — it took me a couple of tries as some of the models listed in there fixed
the cracking _but_ didn't light up my keyboard's LEDs for cases where the mic/sound was enabled.

You could play with `modprobe` to set that module's `model` parameter temporarily but that would also require you to restart `pulseaudio.service` and `pulseseaudio.socket`
using `systemctl` and reconnect your headphones for each attempt. Alternatively, you can just define it permanently (the file described below), reboot and see if it worked well.
If not, tweak `model` accordingly.

The solution in my case was about adding a `*.conf` file (you can name this whatever you want, I named it `sound.conf`) to `/etc/modprobe.d/` that had the following contents:

```bash
options snd-hda-intel model=thinkpad
```

Hope this helps!
