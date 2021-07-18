---
title: 'Notify me if no VPN is active'
category: privacy
tags:
  - VPN
  - Linux
image: /assets/images/2020/router.jpg
---
I have [blogged about ensuring you connect to a VPN](/privacy/2019/12/27/network-editor/) when
connecting to certain wi-fi networks. I have also [blogged about connecting to a VPN when the
computer boots](/privacy/2020/10/14/connect-vpn-whent-computer-boots/). Both of those methods
have merit, but also some inconveniences, like conflicting VPN profiles, or difficulty connecting
on boot (before the password keyring for the VPN password is ready), etc.

I simplified my solution quite a bit, and I made it more resilient.

<!-- more -->

One of the things I said in my previous post [I want a GNOME notification when not connected to VPN ](/notes/2020/03/12/gnome-vpn-notification/):

> I would like to have a GNOME extension that pops a notification whenever I am not connected to a VPN.

Today I made that myself, and it was quite simple.

I started making a script that checks if there is a VPN network interface active. If not, then it uses `notify-send` to
send a notification. Finally, I run that check every 5 minutes using cron.

![The notification](/assets/images/missing-vpn-notification.png)

## How to do it

This is the script `check-vpn-status.sh`:

```shell
#!/bin/bash

if [ $(/usr/sbin/ip address|grep "[0-9][0-9]*: tun[0-9][0-9]*: "|wc -l) -eq 0 ]; then
	notify-send -u critical "VPN connection not detected." -a OpenVPN;
fi
```

I have seen that `notify-send` sometimes does not show notifications when running from cron. I have seen that in
Ubuntu. If that is your case, try adding `XDG_RUNTIME_DIR=/run/user/$(id -u)` at the end.

```shell
#!/bin/bash

if [ $(/usr/sbin/ip address|grep "[0-9][0-9]*: tun[0-9][0-9]*: "|wc -l) -eq 0 ]; then
	XDG_RUNTIME_DIR=/run/user/$(id -u) notify-send -u critical "VPN connection not detected." -a OpenVPN;
fi
```

Then type `crontab -e` to edit your cron jobs and add the following line:

```shell
*/5 * * * * /bin/bash /home/e0ipso/.config/check-vpn-status.sh
```

Of course, you will need to change `/home/e0ipso/.config` to the actual location of your script.

That's it!

<small>Photo by <a href="https://unsplash.com/@extaf_ms?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Misha Feshchak</a> on <a href="https://unsplash.com/s/photos/cybersecurity?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></small>
