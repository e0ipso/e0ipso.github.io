---
title: SteelSeries Arctics 7 headset was too quiet
categories:
  - notes
tags:
  - Personal
image: /assets/images/2020/headset.jpg
---
A couple of weeks ago my 1 year old daughter broke my headset. I had to wear it with tape and it would catch in my hair
all the time, so I decided to get a new pair. After doing my research I settled with the SteelSeries Arctics 7 headset.

<!-- more -->

This new headset is very nice. Some of the features include:

  - It has a decent range when going wireless.
  - It manages battery very well if you forget to turn it off.
  - The quality of the sound is nice (at least for my untrained ear).
  - It has a retractable unidirectional microphone, that helps to isolate from background noise.
  - It has a hardware mute button with a mute indicator I can see. This is a deal breaker for me.
  - It is quite comfortable to wear, and it's not very heavy.

However, it has an annoying defect. It is too quiet, even with the volume up to the maximum. After a bit of online
research I learned that this is a common misfortune among the owners of this product. This blog post is about how I
fixed it in linux.

Open a terminal and execute:

```
alsamixer
```

This will bring up a terminal interface to adjust your sound settings in Linux. The first thing you will want to do is
to select the SteelSeries Arctis 7 sound card (the USB dongle) in the interface. To do that, press `F6`.

![Alsamixer 1](/assets/images/alsamixer1.png)

Once you have selected it, increase the volume. I recommend you to do some testing playing some music, videos, etc.

![Alsamixer 2](/assets/images/alsamixer2.png)

Once you are satisfied, escape Alsamixer using the `ESC` key. You have adjusted the volume now, and that annoyance is
gone. However, if you reboot your computer you will need to re-set your audio volume with `alsamixer`. What you need to
do is to save your setting. To do so you need to use:

```
sudo alsactl store
```
