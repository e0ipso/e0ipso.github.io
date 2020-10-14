---
title: 'Connect to ProtonVPN when computer boots'
category: privacy
tags:
  - VPN
  - ProtonVPN
  - Linux
image: /assets/images/2020/router.jpg
---
I have [blogged about ensuring you connect to a VPN](/privacy/2019/12/27/network-editor/) when
connecting to certain wi-fi networks. I have used this technique to ensure that I stay connected to
my ProtonVPN when browsing the internet. This is an easy step towards online privacy. Recently I
have changed my setup for something even more reliable. This will even protect me against IPv6
leaks.

<!-- more -->

The method that I was using relied on GNOME's [network manager](https://wiki.gnome.org/action/show/Projects/NetworkManager/Features)
to connect to my OpenVPN profile whenever I tried to connect to a certain wi-fi network. Once I set
up my home wi-fi in my desktop computer, I was all set. However, this had some minor drawbacks:

  1. I had to connect manually to the wi-fi after booting. This was due because the password for my
  [ProtonVPN](https://protonvpn.com) was in GNOME's keyring. The keyring was not initialized when my
  system tried to connect to the wi-fi on boot.
  1. Sometimes connection would take a long time waiting for the VPN.
  1. I had the risk of an [IPv6 leak](https://protonvpn.com/support/prevent-ipv6-vpn-leaks/).
  Fortunately my ISP will not provide me with an IPv6 address, so I have not been vulnerable to
  this. Of course, this could change without a warning.

These are minor annoyances that pushed me to research for a better way to stay connected to my VPN
without some/all of those issues.

My final set up involves the ProtonVPN CLI program for Linux, and some systemd scripts.

## 1. ProtonVPN CLI setup
First I installed the ProtonVPN CLI using [the official documentation](https://protonvpn.com/support/linux-vpn-tool/).
The documentation also includes instructions on how to set up the CLI. This is very well detailed
there so I will not go into details. Just a reminder, when providing your username and password you
should NOT use the credentials to log-in to the ProtonVPN.com site. Instead, look in the _Account_
section in your profile and you'll find the username and password for OpenVPN use.

Once I installed everything correctly I could to type:

```
sudo protonvpn connect --fastest -p UDP
```

This produced the following output:

> Connecting to ES#6 via UDP...
>
> Connected!

To ensure that the connection didn't drop immediately after connecting, I typed:

```
sudo protonvp status
```
## 2. Systemd setup
I did not want to be able to connect from the command line, I wanted to make sure I was connected to
the VPN during my computer's start up sequence. For that I turned to `systemd`.

I created three scripts that I copied from various sources with the following paths and contents.
You can download these scripts below.

### Connect to the VPN when booting up the computer
In [`/etc/systemd/system/protonvpn-autoconnect.service`](/assets/files/protonvpn-autoconnect.service):

```
[Unit]
Description=ProtonVPN-CLI auto-connect
Wants=network-online.target

[Service]
Type=forking
ExecStart=/usr/local/bin/protonvpn connect -f
Environment=PVPN_WAIT=300
Environment=PVPN_DEBUG=1
Environment=SUDO_USER=[YOUR-USERNAME-HERE]

[Install]
WantedBy=multi-user.target
```

### Disconnect from the VPN when suspending
If the computer enters suspension, for instance when you close the lid of your laptop. I wanted
to disconnect from the VPN.
In [`/etc/systemd/system/protonvpn-disconnect.service`](/etc/systemd/system/protonvpn-disconnect.service):

```
[Unit]
Description=ProtonVPN-CLI disconnect before sleep
Before=suspend.target
Before=hibernate.target
Before=hybrid-sleep.target

[Service]
Type=forking
Environment=PVPN_WAIT=300
Environment=PVPN_DEBUG=1
Environment=SUDO_USER=[YOUR-USERNAME-HERE]
ExecStart=/usr/local/bin/protonvpn disconnect


[Install]
WantedBy=suspend.target
WantedBy=hibernate.target
WantedBy=hybrid-sleep.target
```

### Re-Connect to the VPN when waking up the computer
After waking up from suspension, I wanted to connect back to the VPN.
In [`/etc/systemd/system/protonvpn-reconnect.service`](/etc/systemd/system/protonvpn-reconnect.service):

```
[Unit]
Description=ProtonVPN-CLI reconnect after sleep
Requires=network-online.target
After=suspend.target
After=hibernate.target
After=hybrid-sleep.target

[Service]
Type=forking
Environment=PVPN_WAIT=300
Environment=PVPN_DEBUG=1
Environment=SUDO_USER=[YOUR-USERNAME-HERE]
ExecStart=/usr/local/bin/protonvpn c -f


[Install]
WantedBy=suspend.target
WantedBy=hibernate.target
WantedBy=hybrid-sleep.target
```

### Enable the systemd services
Before enabling the scripts make sure to change `[YOUR-USERNAME-HERE]` with your username. Also make
sure that `/usr/local/bin/protonvpn` is where ProtonVPN is installed. You can do so with
`which protonvpn`.

Once you have checked that execute these commands to tell systemd to use the scripts:

```
sudo systemctl daemon-reload
sudo systemctl enable protonvpn-autoconnect.service
sudo systemctl enable protonvpn-disconnect.service
sudo systemctl enable protonvpn-reconnect.service
```

## 3. Reconnect every hour
The systemd scripts will connect to the fastest ProtonVPN server _at the time of connection_. That
may vary during the day. Since I do not want to get stuck in a server that has become slow, only
because it was fast when I first connected to it I decided to disconnect and re-connect every hour.
For that I created a root cron job.

```
sudo crontab -e
```

Then add this in a new line:

```
@hourly /usr/local/bin/protonvpn disconnect && /usr/local/bin/protonvpn connect -f
``` 

## Add a visual queue in GNOME
Since I want to know that I am connected by looking at my GNOME bar I installed
[this GNOME extension](https://extensions.gnome.org/extension/3133/protonvpn-status/). It worked
without issue for me.

## Sources
I found the systemd scripts in:
  - [The official documentation for the CLI tool](https://github.com/ProtonVPN/linux-cli/blob/master/USAGE.md#via-systemd-service).
  - [A reddit post for the ProtonVPN subreddit](https://www.reddit.com/r/ProtonVPN/comments/i2f7j5/solution_to_reconnecting_protonvpn_after/).

<small>Photo by <a href="https://unsplash.com/@extaf_ms?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Misha Feshchak</a> on <a href="https://unsplash.com/s/photos/cybersecurity?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></small>
