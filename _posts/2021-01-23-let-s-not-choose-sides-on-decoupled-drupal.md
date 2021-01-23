---
layout: post
title: "Let's not choose sides on Decoupled Drupal"
date: 2021-01-23 19:12 +0100
categories:
  - notes 
tags:
  - drupal
---
Yesterday I had an interesting conversation about the principles of decoupled Drupal with Gabe Sullice in the context of menus. We agree that decoupled Drupal should not be only about websites, there are plenty solutions for that already. However, we hold slightly different visions on how to go about this.

What is your view?

I pasted the Slack conversation here in case anyone is interested in it.

<!-- more -->

---

**gabesullice**

I'm writing an issue to add authenticate and logout links to JSON:API responses. But I can already anticipate a bikeshed over which URL to put in there.

https://drupal.slack.com/archives/C016N3HTHRD/p1611328043014700?thread_ts=1611262827.003000&cid=C016N3HTHRD

> e0ipso
> 
> We should care beyond web developers in this initiative.
> From a thread in #decoupled-menus-initiative

If you have any idea about how to make it flexible without making it complicated, LMK

**e0ipso**

I think that we should be explicit where Drupal is:
  - Web only
  - Web favoring
  - Consumer agnostic

Ideally we should be consumer agnostic, but I fear that's a lost battle already
I find myself constantly wanting to post a reminder about it and I sense only annoyance at that fact. Probably rightly so 😛

**gabesullice**

I'm not annoyed and I don't think anyone is annoyed 🙂

**e0ipso**

I think the echo chamber effect is huge in this case we are a community of people that make a living building websites, and our mental model will keep falling back to that.

**gabesullice**

It would help me understand/share your concern if you could give some examples.

**e0ipso**

My concerns are that when we start favoring the web there is no coming back.

**gabesullice**

Is there a difference between favoring HTTP and favoring the web?

**e0ipso**

Yes.

The web is HTTP + HTML + a Browser. To remain consumer agnostic, we should stay at HTTP IMO.

Do you see it differently?

**gabesullice**

I don't think there's a difference. Browsers are just scriptable HTTP clients, and I think it's okay for us to provide extra goodies for the most widely used HTTP client in the way that many companies provide "official" libraries and "community" libraries in less-used languages.

**e0ipso**

thinks

**gabesullice**

It's why I'm so bullish on cookie auth, for example. It's an HTTP spec. Not a browser spec, even though that's where they're most widely used.

**e0ipso**

I don't think I agree with Browsers are just scriptable HTTP clients. My main objection is with the keyword just.

**gabesullice**

nods

**e0ipso**

> It's why I'm so bullish on cookie auth, for example. It's an HTTP spec. Not a browser spec, even though that's where they're most widely used.

Again, this is a good example. All browsers have specific guardrails against the perils of cookies and the assumptions devs make around them. You cannot guarantee that in other technologies.

**gabesullice**

But the other mechanisms are no safer, AFAIK.

They all boil down to storing a secret on the client, somewhere.

**e0ipso**

Not arguing against that, but you'll have to admit that cookie auth is unique in having all HTTP requests originating from a client -to a host-
attaching the credentials without the user, or the developer, doing anything. Even [when doing] `<img src="http://bankofamerica.com/donate-to-gabe">`. Not saying there isn't a good reason but this seems to be almost unique to browsers

➕1



**gabesullice**

connect the dots for me.

**e0ipso**

`.........`

`---------`

**gabesullice**

lol

I totally agree with what you just said, that the security concern w/ attaching the secret to every request on non-origin domains is unwise and that it's limited to browsers.
I don't see what implications that has though.

Perhaps you're saying that we should not use cookies and instead use Authorization headers to work around that leaky architecture, but that seems like favoring browsers.
Letting browsers influence the design by making auth more difficult for non-browsers (because cookies are easier IMO) 

**e0ipso**

I don't see that way 🤔

I see it as not taking a position in something that is not standard across technologies, therefore not favoring anything. Suffice to say that I am not saying we should not do it, rather ensure that we own the decision with a big billboard.

I think this is one of two things we never managed to agree upon. That's a good thing, that means that when we reach agreement in other things, that is genuine agreement.

🙂

Also, is it OK if I publish this conversation in my blog?
[yes]


**gabesullice**

I don't think we disagree 😛
I began by saying:

> If you have any idea about how to make it flexible without making it complicated, LMK

I want to make the authentication/logout links not favor cookies (but I think cookies should be the default) maybe that's a contradiction 😛

**e0ipso**

ha!

https://en.wikipedia.org/wiki/Nudge_%28book%29

> Wikipedia Nudge (book)
> Nudge: Improving Decisions about Health, Wealth, and Happiness is a book written by University of Chicago economist Richard H. Thaler and Harvard Law School Professor Cass R. Sunstein, first published in 2008.
> The book draws on research in psychology and behavioral economics to defend libertarian paternalism and active engineering of choice architecture.The book received largely positive reviews. The Guardian described it as "never intimidating, always amusing and elucidating: a jolly economic romp but with serious lessons within." It was named one of the best books of 2008 by The Economist.

According to this book, as a choice architect, there is nothing more powerful than defaults to favor an outcome.
FWIW we've been exercising libertarian paternalism for a long time now in our little corner of influence, and that is OK. No choice is a choice.

**gabesullice**

I just realized that the User account menu already has log in/out links. If we can attach menus to JSON:API responses, we don't need to have special code to attach authentication links.

[OAuth module](https://www.drupal.org/project/simple_oauth) could override these links or provide its own.

https://git.drupalcode.org/project/drupal/-/blob/9.2.x/core/modules/user/user.links.menu.yml#L6

**e0ipso**

See! It was easy to start with. We just didn't know yet. 😛 
