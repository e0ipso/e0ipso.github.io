---
title: 'Enable HTTPS in GitHub pages'
categories:
  - notes
tags:
  - Privacy
  - Personal
---
Today my friend Andrew Berry pointed out to me that his browser was warning him about this blog
not having HTTPS enabled. For additional irony he noticed that while reading my article on
[_How to use HTTPS in your local environment_](https://mateuaguilo.com/web-development/drupal/2020/11/24/custom-certificate-generation/).

<!-- more -->

I host this site on GitHub pages as static HTML + JS + CSS. Then I self-host other privacy
respecting services in my server for things like [comments](https://commento.io) and
[analytics](https://matomo.org/). My Jekyll plugin for the feed generation was detecting the URL as
`http://mateuaguilo.com` from the GitHub Pages configuration. This lead to the
[RSS feed](https://mateuaguilo.com/atom.xml) to contain all links as HTTP 😱, even if I added HTTPS
a while ago.

I logged-in to the repository and navigated to the GitHub Pages settings. Now there is an option
to enforce HTTPS. This fixed the RSS feed.

![GitHub pages settings for this blog's repository](/assets/images/github-pages-https.png)
