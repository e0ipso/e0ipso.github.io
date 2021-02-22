---
title: 'How I fixed, "Qt apps are too small in GNOME"'
categories:
  - notes
tags:
  - linux
---
I am a GNOME user, but I still use fantastic apps written using the Qt framework. However in my HiDPI screen (retina screen for macOS people) the Qt apps were rendering too small. They were not applying the scaling factor I requested in my GNOME settings.

<!-- more -->

![Scale factor in GNOME](/assets/images/gnome-scale-factor.png)

In order for Qt apps to have the correct size I created a script `/etc/profile.d/hidpi.sh`. I created it by executing:

```
sudo echo 'export QT_AUTO_SCREEN_SCALE_FACTOR=1' >> /etc/profile.d/hidpi.sh
```

After a computer restart, my Qt applications were showing correctly as usual. I did this in my Fedora 33 Workstation.