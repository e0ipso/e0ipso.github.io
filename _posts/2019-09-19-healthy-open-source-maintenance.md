---
title: Healthy Open Source Maintenance
categories:
  - open-source
tags:
  - Free Software
  - Community
canonical_path: https://www.lullabot.com/articles/healthy-open-source-maintenance
---
_This is a re-post of the article I wrote for the [Lullabot blog](https://www.lullabot.com/articles/healthy-open-source-maintenance)._

Open source has won! It powers software everywhere. From automated irrigation software to supercomputer kernels. This has enabled the industry to evolve at a vertiginous pace that is changing the world.
<!-- more -->
This has all been possible because developers around the globe can now use software for free, and learn from the source code. Developers are not rewriting the JPEG compression library; instead, they are solving more high-level problems because that problem has been solved and liberated.

An initial thought was that this may seem like a utopia come true. Knowledge flowing freely and rapidly between developers around the globe that helps each other to achieve more and more impressive goals. These goals are often funded by private companies that produce software as their main source of revenue. Many small software firms rely entirely on FOSS at all the levels of the organization, so they can avoid fees and licenses that could jeopardize their viability. In other words, capitalism likes open source.

## Giants on the shoulders of mice

Whether we like it or not, all software companies rely on open source. In fact, if you were to write that latest product or library from scratch, without a hint of open source, it would take your company **way** more time, or you would need to spend a lot more in licenses. In any case, your estimations would balloon, and you would lose clients very fast. There is no way back from it; you either use FOSS, or you are out of business.

But how is this dynamic possible? How can organizations do more that costs less? The key is the humans that write this free software, the open source maintainers. “[The Internet Relies on People Working for Free](https://onezero.medium.com/the-internet-relies-on-people-working-for-free-a79104a68bcc)” provides more context into this dynamic. In that article, you can read the experience of [Daniel Stenberg](https://daniel.haxx.se/), maintainer of the uber-ubiquitous cURL for more than 20 years in his spare time, while having a full-time job.

Huge corporations depend on the work of hundreds of open source maintainers. I am one of those, and I occasionally get the question: *Why do you do it?*

In my life, I have done it for different reasons:

- [Because I can](https://dri.es/the-privilege-of-free-time-in-open-source).
- To get more exposure and reach out for a better job.
- To be appreciated.
- To have fun doing it.
- To become a more practiced developer.
- To improve the ecosystem that my employer uses.

Those are my reasons, but other maintainers could have different ones. Like:

- To have other users help maintain a library they use at work.
- Because it's their full-time job.
- To belong to a community that shares their hobbies.
- To share domain knowledge.
- To promote the adoption of a new protocol, framework, etc.

Whatever the reasons are, many organizations need these maintainers to stay motivated. That is because these organizations need maintainers to fix bugs, create new features, improve performance, stay up to date with new framework or language versions, keep up with dependencies, etc.

I asked Alex Pott, core committer of the Drupal project: *In your opinion, is the current sponsoring model sustainable for all the open source maintainers?* He replied:

(All quotes reproduced as written.)

> No not at all. It’s a very big question you are asking and there are many parts to it. I don’t think there is a single current model but if by “current sponsoring model” you mean the current smorgasbord then speaking from my situation - I’m lucky I have 2 companies the pay for time and give me leeway to prioritise my time whilst both of them still require me to do direct work for them. But as a contractor they can cancel my contract at any time so it doesn’t feel a very secure position. Having multiple contracts mitigates the risk a bit and I feel very lucky to have been able to make that happen. In the past I have tried using crowd-sourcing (gittip) and whilst that was initially successful it requires the maintainer to become good at marketing themselves which they might not be good at.
> 
> Also I think you need to define what you mean by sustainable is. Drupal, for example, has a problem with recruiting new core maintainers. There are many reasons for this: the complexity of core code and process, the lack of funding to pay someone for their time unless they work for Acquia, if they do work for Acquia then the perception that Acquia is “taking over” and lots of other reasons.
> 
> I feel that this is a very incomplete answer to your question and I’m happy to discuss more.

Alex’s role is crucial for one of the biggest open source projects in the world. He has been the most active committer for the last six or seven years. Yet he describes his situation as “lucky to have someone that pays” and “insecure”. Alex refers to the “current model” as a [smorgasbord](https://www.wordnik.com/words/smorgasbord), which hints that we lack the structure to ensure proper funding of some important open source maintainers. Alex feels lucky; that may be because the vast majority of open source contributions are not paid.

## Working without a paycheck

“[Getting Paid for Open Source Work](https://opensource.guide/getting-paid)” provides a good insight about the relationship between open source maintainers and money. I will point out that writing open-source software is not like volunteering at a local charity. Maintainers are donating their time for anyone to benefit, **even causes they may not like**. What is more interesting to me is what it's called: the responsibilities of an open source maintainer.
[Nadia Eghbal](https://twitter.com/nayafia) researches the economics of open-source software. She [writes](https://increment.com/open-source/the-rise-of-few-maintainer-projects):

> When a developer named Ayrton Sparling disclosed the presence of malicious code in a popular npm module, event-stream, the response was disbelief—not because the code existed, but because of the way it got there.
>
> The code, intended to steal users’ bitcoin wallets, had been injected by an unknown developer with the username right9ctrl. That person had gained commit access from event-stream’s author, Dominic Tarr, simply by asking for it.
>
> […]
>
> People were baffled by Tarr’s behavior. “You put at risk millions of people, and making something for free, but public, means you are responsible for the package,” wrote one user, XhmikosR.
>
> But Tarr was unapologetic, offering an explanation that seemed maddeningly sincere: “He emailed me and said he wanted to maintain the module, so I gave it to him.”

Dominic Tarr had lost interest in maintaining the project, as he expressed [in GitHub](https://github.com/dominictarr/event-stream/issues/116):

> I don't get any thing from maintaining this module, and I don't even use it anymore, and havn't for years.

And in a subsequent response to the same issue, Tarr adds:

> If you guys feel strongly about this, why don't you volunteer to maintain it and contact npm support?

Tarr was unapologetic because he did not feel obligated to pour his free time into a project against his will, just so users and corporations could keep getting software for free. He passed along maintenance to the only person that volunteered to help. His professional integrity was attacked as a result. He got a lot of negative exposure, perhaps counteracting the time he donated for years. Keep in mind that Tarr maintains many other very successful projects that claim his time as well. All this happened because he wanted to stop, and chose the only person that offered to help. This person was a thief.

It's also important to notice that many software developers are transitioning from a scenario where their companies paid very expensive fees for software. That taught them to expect rapid support responses, high standards of process, etc. They are now in disbelief seeing that Tarr just gave away that critical piece of software they rely upon.

Every reader probably has an opinion of what Tarr should have done differently if anything. And that is the key point of this article.

## Setting expectations

There is a huge mismatch between a Drupal developer, Jane, in a small agency, deciding to contribute some code she wrote for a client, and a full-time sponsored Drupal contributor, Patty. The former will do her best to add some tests, and will tentatively tag a beta release. Like many modules, it will probably stay like that. The latter will:

- Make sure to implement every feature following the recommendations of the different subsystem maintainers.
- Add different types of tests: unit, kernel, and functional.
- Ensure there is high test coverage.
- Respond to bug reports and questions in less than 24 hours.
- Create a test for every bug that gets fixes.
- Write detailed documentation about the new module.
- Promote the module in social media and write a blog post about it for her company blog.
- Record video tutorials.
- Create a public roadmap setting the goals of the module.
- Write detailed notes for every new release.
- Maybe present about the module at conferences.

The reality for these two maintainers is very different. They are both providing goods (the code) and a service (maintaining the project) for free to the end-user. Jane is likely to feel guilty because issues are piling up and she can't find the time for them. She may not even use the module anymore—like Tarr—but she keeps maintaining the project regardless. When she finds the time, she fixes the bug and realizes that she is supposed to write tests, then documentation, then detailed release notes, etc. If Jane is like me, she may not enjoy any of those tasks; she may prefer to play with the kids. She skips these undesired tasks and thinks, “well, I'm sure they'll appreciate I fixed the bugs. Someone can come and write the necessary tests, docs, etc.”

I call these undesired tasks maintenance taxes. Often, I feel like this: *I did more than enough given my availability; I'm sure the users will be happy to split the taxes.*

Patty, on the other hand, doesn't have to pay any taxes. For her, it's *just* work. Nine-to-five work. She takes pride in her craft, and then, after work, she plays the piano.

These two modules will coexist in the Drupal ecosystem, and users are likely to miss the background. Even experienced users—even other maintainers—may expect that they will get free support from the maintainer in a short time frame. Users expect enterprise-grade maintenance for free. Some are convinced that it's the maintainer's obligation, and softly demand this is an open source user's right.

I asked [Cristina Chumillas](http://vermell.cat), a UX maintainer of the Drupal project (among others), the following question for this article: *Who gets the bigger piece in the current economic paradigm with the open source movement? What is the role of the maintainer in this model?* She replied:

> The biggest beneficiaries in the Open Source world are usually the big players: those who have developed and released the product or the main maintainers who sell related services.
> 
> The benefits for the main actors can go beyond money giving corporate value attached to a project, so smart companies pay full-time contributors to evolve the project and help it become more popular. The side effect of this is that the big players are the ones deciding the direction of the product: the features that better suit their product are the ones that will receive more attention by the payed contributors, and the ones that will evolve over time.
> 
> It isn't bad per se, we just need to be realistic about the goals behind it and not idealize a model that has big actors behind it: Google or Microsoft have released several products under Open Source Licenses like vscode or Kubernetes. This shapes the industry and its communities, and although concepts like meritocracy or free contributions are still strongly related to Open Source for many, forgetting the reality can lead to fallacies.

It seems fair to derive from Cristina's response that Patty is more likely to have an influential impact on the project. That, added to the fact that she pays less in taxes, could demotivate Jane from contributing further. We need to make Jane's contributions more enjoyable for her, or we risk losing her highly valuable contributions.

## The Healthy Maintainer Manifesto

I maintain many different modules on Drupal.org. Several of them are quite popular in the small (but growing) decoupled Drupal community. I am grateful that I have always known how to set boundaries, and say "no." I can still improve in that area, but it is a skill that I leverage in my open source life. Other people have other skills but struggle with setting boundaries. They often end up coerced into volunteering more time to serve other developers and corporations.

I asked this question to Jordi Boggiano, maintainer of Composer (among other projects): *Do you have boundaries with the code you give away for free? If so, what are they?*

> I guess what I learned with time is that if I release things for free, of course I have some responsibility to maintain etc, but that's mostly self-inflicted responsibility and I really don't owe anything to anyone. It's important not to let yourself be brought down by users who complain and don't contribute anything. When I don't have time I don't feel bad if things fall behind for a while.
> — Jordi Boggiano

I would love to have an action item to help mitigate that lack of funding that Alex Pott was mentioning. However, I do not know how to help with that. While the economy figures that out, I think we ought to help maintainers not to feel coerced.

My proposal to the Drupal Association and the Drupal community is the following: let's create a special file in the project root of contributed projects called `MAINTAINER_MANIFESTO.md`. This file will set the expectations. Each maintainer is free to write their manifesto, but ideally there will a selection of them readily available to choose with one click. Just like with the license file, or the code of conduct on GitHub. Even though this idea is still in early stages, for the sake of clarity, this is what a selection of manifestos could look like:

- The [*sponsor me*](https://www.jrockowitz.com/blog/sponsor-a-feature) *to get priority on your bug fix* manifesto.
- The *this is free goods and service, so you get what you get* manifesto.
- The *micro-donations to keep the project alive, but I decide what to work on* manifesto.
- The *hire the agency I work for to sponsor features* manifesto.
- …

I imagine the contents of this manifesto to be wildly different for Jane and for Patty. I think that offering a selection of pre-made manifestos is crucial because we don't want yet another maintenance tax.

The presence of the *Healthy Maintainer Manifesto* will be a tool to deflect social pressure. This will help maintainers that suffer from avoidance anxiety because they are not obligated to pay the maintenance taxes. Their manifesto says so! If any demanding user wants to pressure them into doing anything in particular, they can point them to the manifesto. The manifesto replaces complicated social interactions with *a discharge form*. The goal of this is to protect the maintainer, because, after all, maintainers are the cornerstone of open source—and thus all the software industry. And because we all prefer a community where maintainers are happy, and not at the brink of burnout.

## Let's do the experiment!

Aside from writing this article, I am trying to:

- Get an actual list of pre-made manifestos to choose from.
- Raise interest in the different open source registries, starting with Drupal.org, to implement the easy selection of manifesto upon project creation.

If you want to help, leaving [feedback in this issue](https://www.drupal.org/project/drupalorg/issues/3082225) is a fantastic way to help.

*I want to thank [Jordi Boggiano](https://seld.be/about), [Cristina Chumillas](http://vermell.cat/), and [Alex Pott](https://www.drupal.org/u/alexpott) for their time responding questions for this article. Special thanks to [Christian López](https://about.me/penyaskito) for his feedback and encouragement.*
