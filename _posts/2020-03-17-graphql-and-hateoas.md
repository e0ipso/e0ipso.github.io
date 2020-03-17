---
title: HATEOAS vs GraphQL in Drupal
categories:
 - drupal
tags:
 - Open source
 - Drupal Development
---
The other day I had the chance to participate in an interesting thread in the Drupal Slack.

My position in this problem space has been known for quite a while. I think REST
(with [HATEOAS](https://en.wikipedia.org/wiki/HATEOAS)) is a superior technology to GraphQL. This is for several
reasons:
<!-- more -->

  1. REST leverages HTTP. This is a huge advantage, GraphQL has many years of catching up to do and this seems to happen
  but creating a strong dependency on the Apollo client.
  1. HATEOAS simplify the maintenance and increase the flexibility of the front-ends.
  1. GraphQL's main point is on a network hack for HTTP 1.1. This hack is also available for REST, as an additional
  feature. When the adoption of [HTTP/2](https://en.wikipedia.org/wiki/HTTP/2) increases GraphQL will have to reinvent
  itself or become obsolete.

This is the conversation:

> geertdv: Question, is someone using contenta cms with graphql (drupal api)  instead of json api, and if so, what are your findings with it? I've build some gatsby projects in the past utilizing the graphql api directly and looking to implement contenta and its features. 🙂
> 
> benjy: We are using jsonapi but I wish we'd us d graphql, we are just exploring moving certain parts over to the contrib graphql module
> 
> geertdv: And are you using contenta cms or another distro?
> 
> joaogarin: I think contenta includes graphql. Not too sure if many people are or not using it with contenta though..shouldnt be too different than using the module on its own so all the graphql docs should apply. #graphql channel can also be helpful for graphql specific questions
> 
> wimleers (he/him): @benjy what makes you wish that?
> 
> benjy: @wimleers (he/him) main reason is that the support for graphql JS clients is much better. The likes of Relay and Apollo have good architectures, very well supported, pluggable caching, relay can handle aggregating all the fragments your components need into just 1 query etc. We couldn't find anything similar for JSON API so we partly built our own using custom hooks and json-api-normalizer. I think FB backing Relay is a huge advantage, for example, Suspense is announced and Relay is released with a working beta. Happy to discuss more
> 
> benjy: I don't want to criticise the work done for JSON API in drupal, that side of things has been great and i'd use that again, but if I was going 100% headless again, i'd definitely look at the frontend clients and approaches that have the most traction and then either make Drupal supply the data in the right format (e.g. GraphQL) or not use Drupal.
> 
> wimleers (he/him): @gabesullice @e0ipso ^^
> 
> wimleers (he/him): Thanks for taking the time to write up your thoughts, @benjy — much appreciated 🙂
> 
> e0ipso: https://invidio.us/watch?v=vgm_uGmspMI my recommendation is to watch this.
> 
> benjy: @e0ipso what specifically? I've followed Phil for a long time so I know most of what he preaches, i'll watch this tonight.
> 
> benjy: it feels like the normal comparing pro and cons of the api technology where as everything i'm getting at is the quality of the js clients
> 
> wimleers (he/him): @benjy good point — /cc @gabesullice @zrpnr
> 
> e0ipso: @benjy for me the key point is the state transfer
> 
> e0ipso: you either have a good hypermedia strategy
> 
> e0ipso: and use REST
> 
> e0ipso: So you can have solid and highly resilient front-ends
> 
> e0ipso: which is vital for mobile apps where you cannot force push updates
> 
> e0ipso: OR you use REST-ish or GraphQL and deal with the high maintenance costs associated to that.
> 
> e0ipso: We couldn't find anything similar for JSON API so we partly built our own using custom hooks and json-api-normalizer.
> This is a good point. I feel that a client should be lean.
> 
> e0ipso: You should not need caching layers, that's HTTPs work
> 
> e0ipso: unless you have a custom reverse proxy like node.js
> 
> e0ipso: in that case got has you covered 💯
> 
> benjy: @e0ipso well the experience moving between pages is much nicer if the entity / response is cached on the client in redux or a hook etc it's instant
> 
> e0ipso: @benjy I think that the main thing might be the opinionated page-level query builder (Relay)
> 
> benjy: yeah that is a good feature for sure, we've found it's much more reliable if every component loads the data it needs
> 
> e0ipso: which is a good deal
> 
> e0ipso: @benjy if a response is cached in HTTP it means it's in the browser
> 
> benjy: but if you have 3 components on the 1 page all rendering a different field from a node, you don't want to have 3 api calls
> 
> e0ipso: that is instantaneous to load: 0 ms
> 
> e0ipso: you don't want to have 3 api calls
> Yeah, the page level composition from the components definitions is the missing feature indeed.
> 
> benjy: @e0ipso how does the http cache work with mutations though? it wouldn't know if the resource had changed on the server would it
> 
> e0ipso: you get a new response with a patch, afaik
> 
> e0ipso: that carries the new data with the cacheability info for it
> 
> benjy: hmm, and then a follow on GET would be instant because the browser updating the cache after the patch? That isn't something i've experienced but that would potentially solve some issues
> 
> benjy: @e0ipso another issue that we were able to solve with a client side cache is similar to the page composition stuff but your api layer can return the promise for the already sent http request
> 
> benjy: so even though there hasn't been a response yet, all 3 components that made the same api call, get the same promise
> 
> benjy: i guess that would be doable without caching the actual result and relying on the http cache after the first request completes
> 
> e0ipso: My point in giving you a link to the talk is that I think Phil makes a compelling case
> 
> e0ipso: on outlining that REST with HATEOAS is superior tech
> 
> e0ipso: BUT
> 
> e0ipso: if you have a junior team working on the client
> 
> e0ipso: that have to learn on the job
> 
> e0ipso: then GraphQL is a VERY nice pre-packaged solution that get's you out of the immediate necessity
> 
> e0ipso: (although that may lead to increased maintenance cost)
> 
> e0ipso: I'm also curious what @zrpnr and @gabesullice think.
> 
> gabesullice: I think there's definitely something to be said about the "quality" of the available clients. They can cause problems if they're not precisely fit to your needs. However, much of what Apollo does for you is already done for you by various layers of the HTTP stack. Server side caching, CDNs, browser caching, cache validation with Etag/If-None-Match etc. It's all highly composable.
> Personally, I never use any of the clients listed on jsonapi.org. fetch is good enough for me. I know lots of people really like Axios (although I dont understand why).
> When it comes to junior devs, the reasoning that they're somehow better of using GraphQL doesn't ring true to me. GraphQL is a query language and accordingly implies that they're having to learn a new query language.  How is learning HTTP semantics harder/less important?
> 
> gabesullice: I say all this hoping that you'll take it with a gigantic bucket of salt. I'm obviously a really, terribly biased person to talk to about it 😛
> 
> benjy: Thanks for everyone's input always good to hear.
