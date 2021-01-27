---
layout: post
title: Practical Progressive Decoupling Explained
category: drupal
categories:
   - web-development
   - drupal
tags:
   - Progressive Decoupling
   - Decoupled Drupal
   - Drupal Development
image: /assets/images/2021/land.jpg
---
I recently recorded a video series tutorial about progressive Drupal decoupling. In this series I
take two of the official React app examples and turn it into a _widget_. You Drupal editorial team
can then embed those React applications (a calculator, and an emoji selector) as blocks in a page, as
a field in a content type, as an embedded entity in the body field using the WYSIWYG, ...

<!-- more -->
## #1 Embed Any JavaScript Application
In this first video in the series we will take one of the offical examples from react and we will turn it into a widget ready to be embedded in Drupal (or anywhere else).

Steps
1. Create new repository from template.
2. Migrate source to new repo.
    - Copy new source files.
    - Adapt index.js (including render function).
    - Combine package.json.
    - Find & replace «widget-example».
    - Remove / add specific features.
3. Reformat and execute tests.
4. Execute locally.
5. Deploy application.

<div class="video-wrapper"><iframe allowfullscreen src='https://youtube.com/embed/dYaVNL7xmB0'></iframe></div>

## #2 The Registry & the App Catalog

The widget registry is the place where you aggregate your widgets (and other people's widgets you want to use) to make them discoverable to Drupal and other CMS integrations.

This piece plays a fundamental role in the governance of your project(s). You can choose to have a single registry for all your Drupal installations, or one registry per project. You can use the pull requests to gatekeep what versions are added to the registry and who can publish them. The idea is that the owner of the widget-registry project has the authority of accepting PRs to add/update widgets so they are available in the registry (and therefore in Drupal).

<div class="video-wrapper"><iframe allowfullscreen src='https://youtube.com/embed/pNHNGs_W0cs'></iframe></div>

## #3 Set up Progressive Decoupled Drupal
In this video we will learn how to connect Drupal and the widget registry to let editors embed JS applications all over Drupal (that includes support for i18n!).

You can, for instance, embed JS applications as blocks, as a field for a content type, in the body field as an entity embed, ...

<div class="video-wrapper"><iframe allowfullscreen src='https://youtube.com/embed/LX808DtJB8Y'></iframe></div>

<small>Photo by <a href="https://unsplash.com/@sotti?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Shifaaz shamoon</a> on <a href="https://unsplash.com/collections/4687121/increment-collection?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></small>
