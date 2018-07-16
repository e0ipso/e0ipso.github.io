---
title: Introducing Contenta JS
categories:
 - web-development
tags:
 - nodejs
 - Contenta CMS
 - Decoupled Drupal
 - Drupal Development
---
It seems it was yesterday, but [Contenta CMS got the first
stable](https://medium.com/@mateu.aguilo.bosch/contenta-cms-reaches-1-0-27cfcc3c70c6) release more than a year ago. In
the mean time we have started using Media in core, we improved Open API support, provided several fixes for the Schemata
module, written and introduced JSON RPC, and made plans to transition to the Umami content model from Drupal core. A lot
has happened behind the scenes.
<!-- more -->
[Contenta CMS](https://github.com/contentacms/contenta_jsonapi) is being used as is and it's inspiring the custom
development of decoupled Drupal applications. Both of these things were primary goals for the project. But many times
Drupal, and hence Contenta CMS was only one part of the back-end. [You need a nodejs
back-end](https://github.com/contentacms/contentajs#why)  in front of Drupal in almost any decoupled project. That is
why we started working on a nodejs starter kit for your decoupled Drupal projects. We call this [**Contenta
JS**](https://github.com/contentacms/contentajs).

Until now, each agency had their own nodejs back-end template that they used and evolved in every project. There has not
been much collaboration in that space. This is not the Drupal way. **Contenta JS** is a nodejs project that brings
consistency to this space.

The vision for this project is to create a set of common practices so agencies can collaborate on creating the best
software possible in nodejs, just like we do with Drupal. Through this collaboration we will be able to get features
that we need in every project, for free. Today Contenta JS already comes with many of these features:
  
  - Automatic integration with the API exposed by your Contenta CMS install. Just provide the URL of the site and
    everything is taken care of for you.
      - JSON API integration.
      - JSON RPC integration.
      - Subrequests integration.
      - Open API integration.
  - Multi-threaded nodejs server that takes advantage of all the cores of the server's CPU.
  - A Subrequests server for request aggregation. Learn more about [subrequests](https://github.com/contentacms/contentajs/blob/master/docs/subrequests.md).
  - A [Redis](http://redis.io) integration via the optional [@contentacms/redis](https://github.com/contentacms/contentajsRedis).
  - Type safe development environment using [Flow](http://flow.org).
  - [Configurable CORS](https://github.com/contentacms/contentajs/blob/master/config/default.yml#L66-L85).

![Contenta JS architecture diagram](https://d2mxuefqeaa7sj.cloudfront.net/s_9C6EFB25C38FFE2EEE4263F56712CD754A85B31A07F2F90A3E1E7CEFD5CEEDCB_1531762042830_contentacms-node.png)

Watch the introduction video for **Contenta JS** (6 minutes).

<iframe src="https://www.youtube.com/embed/6bdbqo2tETg?rel=0&amp;showinfo=0" frameborder="0" allowfullscreen></iframe>

If we collaborate under the same space we can come up with new modules that do things like *React server-side rendering
with one command*, or *a Drupal API customizer*, or *aggregate multiple services in a pluggable way*, etc. If this is
something you are passionate about and want to collaborate on it, join the
[#contenta](https://drupal.slack.com/messages/C5A70F7D1) Slack channel. You can also create an issue (or a PR!) in the
[GitHub project](https://github.com/contentacms/contentajs).
