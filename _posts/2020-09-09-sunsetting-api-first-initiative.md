---
title: "The sunset of the API-first initiative"
categories:
  - web-development
  - drupal
tags:
  - jsonapi
  - Decoupled Drupal
  - Drupal Development
image: /assets/images/2020/sunset.jpg
---
TL;DR: we are putting an official end to the [API-first initiative](https://www.drupal.org/project/ideas/issues/2757967) although we don't consider our work done. The initiative leads have struggled to find availability or energy to keep moving it forward. The [upcoming JS components initiative](https://www.drupal.org/blog/state-of-drupal-presentation-july-2020) will likely light this flame back up, and we'll be there to assist and guide in their API endeavors. [This patch](https://www.drupal.org/project/drupal/issues/3170020) removes the initiative from the MAINTAINERS.txt. 

<!-- more -->

## Connection: close

We mark the API-first initiative done. 👋.

However, we are not done. There are so many things we would like to improve in Drupal core. Those include hypermedia support, Open API and schema reliability, non-entity JSON:API resources, OAuth2, etc. Nonetheless, we believe that this is a good stopping point for the initiative. We achieved so much, there are so many unforgettable moments. This initiative has played a huge role into transitioning Drupal to the next 10 years of success! We are so proud of what we have accomplished that it has not been easy to wrap it up.

Let's have HTTP guide us through this decision.

## 207 (Multi-status)

There is no single definitive reason, but the sum of several reasons have lead us to the unanimous decision to call this initiative done.

### 202 (Accepted)

JSON:API was the major goal and crowning achievement of the initiative. It was hard to imagine how much work, stress, and personal time that would take from us. As of Drupal 8.7 we can use JSON:API as part of Drupal core. This was the first time we were tempted to call it done.

### 426 (Upgrade required)

The API-first initiative started with Wim working hard to make REST in core more reliable. In parallel Mateu was writing a myriad of contrib modules bring decoupled Drupal more usable to deliver the expected experiences for customers. In the midst of this Gabe joined the team to help us guide the project to the finish line with the contribution of many volunteers—to whom we are supremely grateful. Thank you! 🙏

It seems this was such a long time ago. Nowadays none of us has the time or the energy to keep tugging this ship. It's time for a different team to take the lead.

### 429 (Too many requests)

We recognize that Drupal has a very long road to truly become API-first. On top of that we still have to finish and polish so many key contributed modules. At different points we entertained the idea of adding to core:

* Consumers.
* Simple OAuth.
* JSON:API Hypermedia.
* Open API.
* JSON:API Resources.
* Decoupled Router.
* Consumer Image Styles.

We probably don't want them all, but we talked about how very useful they are to the decoupled Drupal endeavour. It is also worth mentioning that there are many other modules that are not very good candidates to Drupal core because they are not broadly applicable enough, while they remain tremendously important to the API-first ecosystem. One obvious example: JSON:API Explorer.

### 303 (See other)

Dries proposed a new JS components initiative that will likely begin by "solving" decoupled navigation. We vehemently agree that is a great place to start. It is a thorny problem that is not very well solved right now from the API perspective and requires API-first users to reinvent their own solutions over-and-over.

We expect that the JS components initiative team will have to work on the API side in order to complete their goals. We will be there for them, and we will help with code. We hope to be involved as much as we are able to.

## `Cookie: Contributors=new;`

This post was about marking the initiative as done. As such we have talked about the decisions, and the mindset of the initiative coordinators. However, it would be terribly unfair to add this bow on top of the initiative without mentioning the contributors. Everyone that contributed with code, documentation, ideas, designs, support, etc. THANK YOU.

<small>Photo by <a href="https://unsplash.com/@quinoal?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Quino Al</a> on <a href="https://unsplash.com/images/nature/sunset?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></small>