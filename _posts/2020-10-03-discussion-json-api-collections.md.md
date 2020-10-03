---
title: 'A discussion about JSON:API collections'
categories:
  - notes
tags:
  - drupal
---
The other day I had an interesting conversation about JSON:API collections and how different
projects may benefit from one collection style or another. Participants in the thread were Gabe
Sullice, Matt Glaman, and myself.

I pasted the Slack conversation here in case anyone is interested in it.

<!-- more -->
> **mglaman Sep 25th at 10:36 PM**
>
>
> TIL: a condition filter with IN using ~300+ values causes a 404 ☺
> 
> **gabesullice  3 days ago**
>
> 😱
> 
> **mglaman  3 days ago**
>
> BUT that doesn’t crash the encoding.. locally
> 
> **gabesullice 3 days ago**
>
> The query string might be truncated by your server
> 
> **mglaman  3 days ago**
>
> It might be due to PHP’s built-in server
> 
> **gabesullice  3 days ago**
>
> Don't tell anyone:
> I've been starting to regret JSON:API's entire collections feature and wishing I had implemented a simpler Views for custom collections
> 
> **mglaman  3 days ago**
>
> Yeah but I hate Views
> 
> **mglaman  3 days ago**
>
> Honestly everything has been great, it’s just this new customer has an epic data model
> 
> **gabesullice  3 days ago**
>
> It wouldn't have been Views. It would be much, much simpler.
> 
> **gabesullice  2 days ago**
>
> The current system is super powerful but I think it leads to suboptimal architectures (edited) 
> 
> **gabesullice  2 days ago**
>
> Too much business logic for the FE to handle
> 
> **mglaman  2 days ago**
>
> thinking something like GraphQL’s “cached queries”?
> 
> **gabesullice  2 days ago**
>
> Yeah, closer to that. I'd call them "named collections". The UI would be an interface for creating filters pretty much as they work now (conditions + groups), but they'd be on the server side. One could use JSON:API resources to do completely custom collections too. (edited) 
> 
> **gabesullice  2 days ago**
>
> Anyway... fun thoughts. It's the weekend! 👋
> 
> **gabesullice  2 days ago**
>
> (one last thing @Grimreaper basically built the UI for this already in https://www.drupal.org/project/entity_share)
> Drupal.org
> 
> **e0ipso  1 day ago**
>
> fwiw I have walked that approach in the past
> 
> **e0ipso  1 day ago**
>
> it has lead to suboptimal architectures
> 
> **e0ipso  1 day ago**
>
> and while I see your regrets and how "named collections" can be appealing
> 
> **e0ipso  1 day ago**
>
> I think the grass is actually browner with that approach, not greener
> 
> **gabesullice  1 day ago**
>
> Interesting. What happened? What didn't work?
> 
> **e0ipso  1 day ago**
>
> many consumers have many different data needs
> 
> **e0ipso  1 day ago**
>
> and many times the backend is not even around to do maintenance to deploy updates to their saved collections
> 
> **e0ipso  1 day ago**
>
> and other times those saved collections need to be versionable
> 
> **e0ipso  1 day ago**
>
> because you cannot force updates in all Samsung TVs in the world, like you can do with a JS app
> 
> **e0ipso  1 day ago**
>
> and many other times you have to synchronize deploys of the back-end and 3 different front-ends
> 
> **e0ipso  1 day ago**
>
> (which is a particular type of hell)
> 
> **gabesullice  1 day ago**
>
> How did the consumers locate those named collections?
> 
> **e0ipso  1 day ago**
>
> it varied at different points in the project and across consumer
> 
> **gabesullice  1 day ago**
>
> Like, could you have done something like /collection/v1/name pointed to by a link with rel named-collection-v1?
> I.e. you could deploy a v2 before any consumer "knows" to look for named-collection-v2?
> 
> **gabesullice  1 day ago**
>
> And un-updatable consumers could keep looking for named-collection-v1 forever?
> 
> **e0ipso  1 day ago**
>
> that still requires back-end team to build these into the server
> 
> **e0ipso  1 day ago**
>
> and requires multiple round trips to the server (I know, I know, ...)
> 
> **e0ipso  1 day ago**
>
> IMO that approach works VERY well for a single JS client that is managed by the same team that manages the back-end.
> 
> **e0ipso  1 day ago**
>
> But at that point, I am having trouble recommending a decoupled architecture for such a project.
> 
> **gabesullice  1 day ago**
>
> Fair point but I was thinking is that with the right UI, the frontend team could build them.
> FWIW, I shouldn't have said so strongly that I think fancy filters always lead to suboptimal architectures which implied that named collections lead to good ones.
> But I do think it's impossible to build really great systems without some cooperation between the FE and BE.
> It's always going to be hard to build well-architected systems.
> 
> **gabesullice  1 day ago**
>
> (cooperation at the system level too, not just the teams)
> 
> **e0ipso  1 day ago**
>
> yeah
> 
> **e0ipso  1 day ago**
>
> that tends to contradict almost all the articles about Why decouple?
> 
> **gabesullice  1 day ago**
>
> All the articles are wrong :P
> 
> **e0ipso  1 day ago**
>
> There is always a line where they say: "you can have separate teams that work in separate timelines, in parallel"
> 
> **e0ipso  1 day ago**
>
> Also, Oh! YEAH. My tests finally passed for Simple OAuth 5.x (edited) 
> 
> 
> **gabesullice  1 day ago**
>
> You should "decouple" because html is not flexible enough for the UIs you want to build. Your FE team should focus on making those interfaces and let the BE API "tell" it what to do.
> Or, because you can't use HTML and CSS on a particular platform. In that case you should still let the server tell you what to do at a high level.
> 
> **gabesullice  1 day ago**
>
> IOW, don't decouple business logic. Decouple GUIs
> 
> **e0ipso  1 day ago**
>
> I hear you. My only but is that some times different teams will have different business logic. Gardening that in the back-end becomes tech debt quite fast.
> 
> **e0ipso  1 day ago**
>
> so... I guess that "it depends" on the use case.
> 
> **e0ipso  1 day ago**
>
> @gabesullice @mglaman do you mind if I copy this slack convo in my blog?
> 
> **mglaman  1 day ago**
>
> @e0ipso I don’t mind! I missed most of it 😮
> 
> **mglaman  1 day ago**
>
> I’ll try to read later
> 
> **gabesullice  1 day ago**
>
> Yes, that's okay with me.
