One the privacy higene topic, one cool trick that I learned is that I can configure Gnome to auto connect to a VPN when a certain wifi network is used. This may be available to some distributions in the control panels, but I couldn't find it in mine. To configure this, go to your terminal and type:

```
nm-connection-editor
```

This will bring up a panel like the one in the picture.

![Connection Editor](/assets/images/2019-12-27_10-10.png)

Just select the network, then click the gear, and finally select the VPN you want to connect to. I use this on public wifis and with my home wifi.

![Connection Editor, second screen](/assets/images/2019-12-27_10-11.png)
