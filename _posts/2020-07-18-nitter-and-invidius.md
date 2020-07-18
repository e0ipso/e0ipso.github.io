---
title: 'Stop Twitter and YouTube from spying on you'
categories:
  - privacy
tags:
  - Free Software
  - Privacy
---
During the past years I have changed my digital habits quite a lot. I have deleted many of the
services that I happily used in the past. These include LinkedIn, Facebook, Gmail, Twitter,
Meetup, and many more. Most of those I have never missed, but some of them are so ubiquitous
they are hard to avoid.

<!-- more -->

I get links to tweets and YouTube videos daily. I welcome them. While those platforms
may be **incredibly creepy**, they have the best user-contributed content. Luckily they are
both accessed using a web browser and I have these two
extensions installed: [Invidious redirect](https://addons.mozilla.org/en-US/firefox/addon/hooktube-redirect/),
and [Nitter redirect](https://addons.mozilla.org/en-US/firefox/addon/nitter-redirect/). Both extensions
operate under the same concept, they take you to a privacy minded service that shows the same content
as YouTube and Twitter. These servers run Open Source front-ends for YouTube and Twitter.

The code for these front-ends can be accessed here:
  * https://github.com/zedeus/nitter
  * https://github.com/iv-org/invidious

Both of these have a reference instance/website running the code that anyone can use (and that is what these
extensions will redirect you to by default), but anyone can take that code and host their own instance.
Including you. The reference instances are [nitter.net](https://nitter.net) and [invidio.us](https://invidio.us).

The end result is that I can access the content of Twitter and YouTube, without having to visit their sites.
Additionally, if I click a link to those websites, my browser extensions will notice, and they will take
me to that **same content** in the Nitter/Invidious instance I have configured.