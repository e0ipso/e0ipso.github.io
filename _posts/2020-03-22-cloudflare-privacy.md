---
title: Cloudflare Privacy
categories:
 - notes
tags:
 - Privacy
 - Personal
---
Just a moment ago I was thinking about adding a free Cloudflare CDN in front of my personal services that I self-host at
home.
<!-- more -->
My thinking was on leveraging the edge cache to alleviate some load in my home server. I decided to read [their privacy
policy](https://www.cloudflare.com/privacypolicy#3--how-we-use-information-we-collect) and I was hesitant. This is
an extract:

> we may use the information [...] to:
> [...]
> Process for other purposes for which we obtain your consent.

This is somewhat sketchy, specially in a world where online consent could be asked and granted by "continue to use this
webpage".

> Information from Third Party Services. We may combine information we collect as described above with personal
> information we obtain from third parties. For example, we may combine information entered on a Cloudflare sales
> submission form with information we receive from a third-party sales intelligence platform vendor to enhance our
> ability to market our Services to Customers or potential Customers.

There are some hopeful nuggets in there, like:

> We keep your personal information personal and private. We will not sell or rent your personal information to anyone.
> We will not share or otherwise disclose your personal information except as necessary to provide our Services or as
> otherwise described in this Policy without first providing you with notice and the opportunity to consent.

and

> If we make changes to this Policy that we believe materially impact the privacy of your personal data, we will
> promptly provide notice of any such changes (and, where necessary, obtain consent), as well as post the updated Policy
> on this website noting the effective date of any changes.

Which is refreshing in a world where the common norm is that websites do not notify changes in policies and reserve the
right to make unilateral changes to the original agreement.

I decided not to use Cloudflare after all, because of the small use I would give it and because of [other privacy and
security considerations](https://www.reddit.com/r/privacy/comments/cki0s5/what_makes_cloudflare_bad). Maybe it is a good
choice for you, I haven't made my mind yet for a read-only site.
