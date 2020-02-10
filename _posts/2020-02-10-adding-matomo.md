---
title: Adding Matomo to my Blog
categories:
 - web-development
tags:
 - Privacy
 - Open Source
---
Yesterday I decided to add some analytics to my blog. This was not something
that I decided lightly since web site analytics are a type of tracking. For the
longest time I considered analytics to be against the rights of privacy that
this blog wants to defend.
<!-- more -->
Hypocrisy is considered one of the deepest sins of our time in the western
societies. Nevertheless I was able to reconcile the internal narrative of a
digital world free of behavioral expropriation and online analytics. The answer
to this was _Open Source_.

[Matomo](https://matomo.org/) is a free open source software that has been
around for a long time. It was first know as Piwik, when in 2007 Matthieu Aubry
decided to create an free alternative to Google Analytics. The features of the
project are astounding. There are a lot more features than what I will be using
to understand the popularity of the content in my blog. But more importantly for
me on this topic Matomo includes settings to help respect visitors privacy.

![A section of the privacy settings in Matomo](/assets/images/matomo-privacy.png)

First and foremost it will respect the
[Do Not Track](https://en.wikipedia.org/wiki/Do_Not_Track) header. This is what
browsers like Firefox send to websites to make them know that the user does not
want to be tracked. Most sites, and analytics software, ignore this setting. It
also allows anonymizing the visitors' IP information (and other user information
that is relevant for a read-only blog).

To top that they include tools to apply this settings retroactively to
previously gathered information. Additionally they include the ability to delete
the accumulated data automatically after some time has passed.

After [installing Matomo](https://matomo.org/docs/installation/) in my own
server and enabling it in my blog I was not able to see my own visits. Which was
a bit puzzling until I realized that I am blocking tracking scripts and I have
the _do not track_ setting enabled. Once I tried with a different browser I was
able to see my visit recorded.

![My first recorded visit in Matomo](/assets/images/matomo-visits.png)

