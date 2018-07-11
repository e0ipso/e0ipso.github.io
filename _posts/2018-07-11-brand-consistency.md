---
title: Decoupled back-ends in the age of brand consistency
categories:
 - web-development
tags:
 - nodejs
 - marketing
 - decoupled
 - BFF
 - patterns
---
It may sound surprising to read about brand consistency from a back-end developer. This is traditionally a topic for UX and marketing experts. However, brand consistency is a powerful trend that will affect all the departments in your digital project.

One of the ways I contribute to the Drupal API-First initiative, aside from all the [decoupled modules](https://www.drupal.org/u/e0ipso), is by providing my point of view from the implementation side. Some would call that the real world™ experience with client projects. This means that I need to hold the pragmatic point of view to make sure that we can do with Drupal what clients need from us. While being vigilant on the trends of our industry I have discovered that there is a strong tendency for digital projects to aim for brand consistency, and that will impact on how we do the implementation.

## Brand consistency
When I talk about brand consistency I only refer to a small part of it. Just picture for a moment the home screen of Netflix in your TV. Now picture Netflix on your browser and on the app for your phone. They all look the same, don’t they? This is intentional.

The first time I installed Netflix in my wife’s iPad I immediately knew how to use the app. It took me about a second to learn how to use a complex and powerful application. All this in a device that was foreign to me. I am an Android person but I was able to transition from using Netflix on my phone while on the bus, to my wifes's iPad or the living room TV. I didn’t even realize that I was doing it. Everything was seamless because all the different devices running Netflix had a consistent design and user experience.

If you are interested on the concept of brand consistency and its benefits you can learn more from actual [experts](https://www.youtube.com/watch?v=ke7_89LBgAs) [on the](https://www.campaignlive.co.uk/article/marketing-video-report-mullers-lee-rolston-talks-brand-consistency/1098573) [subject](https://www.clearvoice.com/blog/brand-consistency-why-its-so-important-how-to-achieve-it/). I will focus on the implications on the API design.

## It changes the approach
For the last few years I have been speaking at events and writing about the imperious necessity for your back-end to be presentation agnostic. The reason is that any of the consumers can have radically different data needs. You don’t want your back-end to favor a particular consumer because that will lead to re-coupling, which leads to high maintenance costs for the consumers you turned your back against.

When the UX and designs are consistent across consumers, then the statement *any of the consumers can have radically different data needs* may no longer be true. If they really are consistent, why would the data they need be radically different? You cannot be consistent and radically different at the same time.

Many constraints, API design tips,  and recommendations are based on the assumption of presentation agnosticism. While this is still true for most projects, a significant number of projects have started to require consistency across consumers. So the question is, if we no longer need to be presentation agnostic in our API design, what can we optimize given that we have a single known presentation?

In other words, we made many compromises in order to stay presentation agnostic. What did we give up, and how do we get it back?

## My approach
The first time that I encountered this new situation in a client project my inherent pragmatism was immediately triggered. My brain was flooded with potential optimizations. Together with the rest of the client team I took a breath and started analyzing this new problem space. In this occasion, the client and suggested the [BFF pattern](https://samnewman.io/patterns/architectural/bff/) from the start. This was a great suggestion that we carefully analyzed, and soon embraced.

### What is a BFF?
A BFF, or Back-end For Front-end, is a server-side service that takes care of the orchestration and processing of the different interactions with the API on behalf of the consumers. In short, it does what each consumer would do against your presentation agnostic API, and moves it to a central place in the back-end. The output of the BFF is a JSON object that is ready to be rendered by each of the consumers.

In other words we will build **a consumer in the back-end**, but instead of outputting HTML, CSS, and JavaScript (using the web consumer as an example) we will output a JSON document.

```json
{
  "title": "The screen's title",
  "components": [
    {
      "heading": "I am a component",
      "type": "hero",
      "image": "https://example.org/assets/awesome.jpg",
      "description": "The description is important as well"
    },
    {
      "type": "group",
      "components": [
        {
          "heading": "Our mission",
          "type": "block",
          "description": "We have a mission, and it's important.",
          "image": "https://example.org/assets/icon-1.jpg"
        },
        {
          "heading": "Features",
          "type": "block",
          "description": "The most impressive features out there.",
          "image": null
        }
      ]
    }
  ]
}
```

You can see in the code above that the shape of the JSON response is heavily influenced by the single design and the components in the front-end. This implies some rigidness on front-end differences, but we agreed that’s OK for our case. For your completely different design, the JSON output would look completely different.

### Implementing BFFs
After requirements settle we decide that we will have a single Back-end For Front-end that will power all the consumer applications. Instead of having one BFF for each consumer, as [Netflix used to do it](https://medium.com/netflix-techblog/embracing-the-differences-inside-the-netflix-api-redesign-15fd8b3dc49d), we will only have one. The reason is that with one we ensure brand consistency. Also, as Lee Byron [puts it](https://samnewman.io/patterns/architectural/bff/#comment-2388151981):

> The concern of duplicating logic across different BFFs is more than just maintaining two repositories of similar code rather than one. The concern is the endless fight against accidental divergence.

Additionally, we don’t have those requirements, but the BFF is also the best place to add global restrictions like authentication, request filters, rate limits, etc.

Our team decided to implement this as a set of rigid endpoints in a [Serverless](http://serverless.com) application written in nodejs. As you can imagine, you can implement this pattern with the tools and the stack you prefer. Since this will be so specific to your project’s designs you will likely need to start “from scratch”.

### BFFs on the consumer
We create this consumer in the back-end in order to simplify all the possible front-ends. We move the complexity of building a consumer into a central service that can be reused by all the consumers. That way we can call the consumers, *dumb clients*. This is because the consumers no longer need to craft complex queries (JSON API, GraphQL or else), they don’t need to aggregate 3rd party services, they don’t normalize the data from the different APIs, etc. In fact all the data to render is ready for them in the shape that they need it.

In our particular case we have been able to **reduce the consumers to renderers**. A consumer only needs to:

1. Process an incoming request and extract what screen we need to request to the BFF. Additionally extract any parameters from the request, like the entity ID. In addition to that any global parameters, like the user ID from the device, are added to the parameter bag.
2. With the name of the screen and the extracted parameters the consumer makes a single HTTP request to the BFF.
3. The BFF responds with all the data needed for rendering in a shape ready for rendering. The consumer takes that and renders all the components.
4. The consumer finally adds all the business logic that is exclusive of the front-end on top of the rendered output. This includes ads, analytics, etc.

## Pros and cons
The pros of this approach are stated throughout the document, but to summarize they are:

- Massive simplification of the consumers. Those complex interactions with the API are in a central place, instead of having each consumer team write them, again and again, in their native language.
- Code reuse across consumers. Bug-fixes, changing requirements, improvements, and documentation efforts apply to all consumers since much of the logic lies in the BFF now. 
- Increased performance. The back-end can be optimized in numerous ways since it does not need to enable every possible design. This can mean denormalized documents in Elastic Search with the pre-computed responses, increased cache hit ratios in calls to APIs now that we control how those are made, faster server-to-server communications for 3rd party API aggregation, etc.
- Front-end flexibility. We can ship new features faster when front-ends are *dumb clients* and just render the BFF output. Unless we need to render new components or change they way something is rendered there are few reasons to require an app update. Just think about the last time you updated your smart TV apps to realize how important this is. With this re-coupled pattern we can ship new features to old consumers.

On the other hand, there are some cons:

- Requires a dedicated back-end team. You cannot just install an API generator, like Contenta CMS, that is configured in the UI and serves a flexible JSON API with zero configuration. Now you need a dedicated back-end team to build your BFF. However, chances are that your project already has a dedicated back-end team.
- Brings back the bikeshedding. In DrupalCon Baltimore [I talked about](https://events.drupal.org/baltimore2017/sessions/advanced-web-services-json-api) how the JSON API module stops the bikeshedding. In this new paradigm, we are back discussing things like the shape of the response, the names in it, how to expose these responses, etc.
- It requires cross consumer collaboration. This is because you want to design a BFF that works well for all current consumers and future ones. Collaboration across different teams can be a challenge depending on the organization.
