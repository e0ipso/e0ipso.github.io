---
title: "Decoupled Drupal Hard Problems: Image Styles"
categories:
 - web-development
 - drupal
tags:
 - Decoupled Drupal
 - Drupal Development
canonical_path: https://www.lullabot.com/articles/decoupled-drupal-hard-problems-image-styles
---
_This is a re-post of the article I wrote for the [Lullabot blog](https://www.lullabot.com/articles/decoupled-drupal-hard-problems-image-styles)._

As part of the <a href="https://www.drupal.org/node/2757967">API-First Drupal initiative</a>, and the <a href="http://www.contentacms.org">Contenta CMS</a> community effort, we have come up with a solution for using Drupal image styles in a decoupled setup. Here is an overview of the problems we sought to solve:
<!-- more -->
<ul>
	<li>Image styles are tied to the designs of the consumer, therefore belonging to the front-end. However, there are technical limitations in the front-end that make it impossible to handle them there.</li>
	<li>Our HTTP API serves an unknown number of consumers, but we don't want to expose all image styles to all consumers for all images. Therefore, consumers need to declare their needs when making API requests.</li>
	<li>The <a href="https://drupal.org/project/consumers">Consumers</a> and <a href="https://drupal.org/project/consumer_image_styles">Consumer Image Styles</a> modules can solve these issues, but it requires some configuration from the consumer development team.</li>
</ul>

<h2>Image Styles Are Great</h2>

Drupal developers are used to the concept of image styles (aka image derivatives, image cache, resized images, etc.). We use them all the time because they are a way to optimize performance on our Drupal-rendered web pages. At the theme layer, the render system will detect the configuration on the image size and will crop it appropriately if the design requires it. We can do this because the back-end is informed of how the image is presented.

In addition to this, Drupal adds a token to the image style URLs. With that token, the Drupal server is saying I know your design needs this image style, so I approve the use of it. This is needed to <a href="https://www.drupal.org/forum/newsletters/security-advisories-for-drupal-core/2013-02-20/sa-core-2013-002-drupal-core-denial">avoid a malicious user to fill up our disk</a> by manually requesting all the combinations of images and image styles. With this protection, only the combinations that are in our designs will be possible because Drupal is giving a seal of approval. This is transparent to us so our server is protected without even realizing this was a risk.

The monolithic architecture allows us to have the back-end informed about the design. We can take advantage of that situation to provide advanced features.

<h2>The Problem</h2>

In a decoupled application your back-end service and your front-end consumer are separated. Your back-end serves your content, and your front-end consumer displays and modifies it. Back-end and front-end live in different stacks and are independent of each other. In fact, you may be running a back-end that exposes a public API without knowing which consumers are using that content or how they are using it.

In this situation, we can see how our back-end doesn't know anything about the front-end(s) design(s). Therefore we cannot take advantage of the situation like we could in the monolithic solution.

The most intuitive solution would be to output all the image styles available when requesting images via JSON API (or REST core). This will only work if we have a small set of consumers of our API and we can know the designs for those. Imagine that our API serves to three, and only three, consumers A, B and C. If we did that, then when requesting an image from consumer A we would output all the variations for all the image styles for all the consumers. If each consumer has 10 - 15 image styles, that means 30 - 45 image styles URLs, where only one will be used.

![](/assets/images/screen_shot_2017-10-25_at_5.27.46_pm.png)

This situation is not ideal because a malicious user can still generate 45 images in our disk for each image available in our content. Additionally, if we consider adding more consumers to our digital experience we risk making this problem worse. Moreover, we don't want the presentation from one consumer sipping through another consumer. Finally, if we can't know the designs for all our consumers, then this solution is not even on the table because we don't know what image styles we need to add to our back-end.

On top of all these problems regarding the separation of concerns of front-end and back-end, there are several technical limitations to overcome. In the particular case of image styles, if we were to process the raw images in the consumer we would need:

<ul>
	<li>An application runner able to do these operations. The browser is capable of this, but other more challenged devices won't.</li>
	<li>A powerful hardware to compute image manipulations. APIs often serve content to hardware with low resources.</li>
	<li>A high bandwidth environment. We would need to serve a very high-resolution image every time, even if the consumer will resize it to 100 x 100 pixels.</li>
</ul>

Given all these, we decided that this task was best suited for a server-side technology.

In order to solve this problem as part of the API-First initiative, we want a generic solution that works even in the worst case scenario. This scenario is an API served by Drupal that serves an unknown number of 3rd party applications over which we don't have any control.

<h2>How We Solved It</h2>

After some research about how other systems tackle this, <a href="http://www.youtube.com/c/Mateu-e0ipso">we established that we need a way for consumers to declare their presentation dependencies</a>. In particular, we want to provide a way to express the image styles that consumer developers want for their application. The requests issued by an iOS application will carry a token that identifies the consumer where the HTTP request originated. That way the back-end server knows to select the image styles associated with that consumer.

![](/assets/images/screen_shot_2017-10-25_at_5.34.37_pm.png)

For this solution, we developed two different contributed modules: <a href="https://www.drupal.org/project/consumers">Consumers</a>, and <a href="https://www.drupal.org/project/consumer_image_styles">Consumer Image Styles</a>.

<h3>The Consumers Project</h3>

Imagine for a moment that we are running Facebook's back-end. We defined the data model, we have created a web service to expose the information, and now we are ready to expose that API to the world. The intention is that any developer can join <a href="http://developer.facebook.com">Facebook</a> and register an application. In that application record, the developer does some configuration and tweaks some features so the back-end service can interact optimally with the registered application. As the manager of Facebook's web services, we are not to take special request from any of the possible applications. In fact, we don't even know which applications integrate with our service.

The Consumers module aims to replicate this feature. It is a centralized place where other modules can require information about the consumers. The front-end development teams of each consumer are responsible for providing that information.

This module adds an entity type called Consumer. Other modules can add fields to this entity type with the information they want to gather about the consumer. For instance:

<ul>
	<li>The Consumer Image Styles module adds a field that allows consumer developers to list all the image styles their application needs.</li>
	<li>Other modules could add fields related to authentication, like OAuth 2.0.</li>
	<li>Other could gather information for analytic purposes.</li>
	<li>Maybe even configuration to integrate with other 3rd party platforms, etc.</li>
</ul>

<h3>The Consumer Image Styles Project</h3>

Internally, the Consumers module takes a request containing the consumer ID and returns the consumer entity. That entity contains the list of image styles needed by that consumer. Using that list of image styles Consumer Image Styles integrates with the JSON API module and adds the URLs for the image after applying those styles. These URLs are added to the response, in the meta section of the file resource. The Consumers project page describes how to provide the consumer ID in your request.

```json
{
  "data": {
    "type": "files",
    "id": "3802d937-d4e9-429a-a524-85993a84c3ed",
    "attributes": {},
    "relationships": {},
    "links": {},
    "meta": {
      "derivatives": {
        "200x200": "https://cms.contentacms.io/sites/default/files/styles/200x200/public/boyFYUN8.png?itok=Pbmn7Tyt",
        "800x600": "https://cms.contentacms.io/sites/default/files/styles/800x600/public/boyFYUN8.png?itok=Pbmn7Tyt"
      }
    }
  }
}
```

To do that, Consumer Image Styles adds an additional normalizer for the image files. This <a href="https://www.lullabot.com/articles/using-the-serialization-system-in-drupal">normalizer</a> adds the meta section with the image style URLs.

<h2>Conclusion</h2>

We recommend having a strict separation between the back-end and the front-end in a decoupled architecture. However, there are some specific problems, like image styles, where the server needs to have some knowledge about the consumer. In these very few occasions the server should not implement special logic for any particular consumer. Instead, we should have the consumers add their configuration to the server.

The Consumers project will help you provide a unified way for app developers to include this information on the server. Consumer Image Styles and <a href="https://www.drupal.org/project/simple_oauth">OAuth 2.0</a> are good examples where that is necessary, and&nbsp;examples of how to implement it.

<h2>Further Your Understanding</h2>

If you are interested in alternative ways to deal with image derivatives in a decoupled architecture. There are other alternatives that may incur extra costs, but still worth checking: <a href="https://cloudinary.com">Cloudinary</a>, <a href="http://tech.akamai.com/image_converter/">Akamai Image Converter</a>, and <a href="https://www.ft.com/__origami/service/image/v2">Origami</a>.

<em>Hero Image by&nbsp;Sadman Sakib. Also thanks to <a href="https://twitter.com/da_wehner">Daniel Wehner</a>&nbsp;for his time&nbsp;spent on&nbsp;code and article reviews.</em>
