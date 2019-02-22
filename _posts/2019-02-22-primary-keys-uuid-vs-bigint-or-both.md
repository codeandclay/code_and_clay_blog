---
layout: post
title: 'Primay keys: UUID vs bigint – or both?'
---

Some tables in the project I'm working on have UUIDs as primary keys others are indexed by `bigint`. To have both wasn't a conscious decision. I began by choosing UUIDs for some tabels because I expected I'd need to expose those ids in the URLs. I intended to use UUIDs throughout the model but I forgot and ended up with some tables being keyed by integers.

I wondered if it was a bad idea to have primary keys of different types, and I came across this [post](https://tomharrisonjr.com/uuid-or-guid-as-primary-keys-be-careful-7b2aa3dcb439). There's plenty to take in and I'll need to read it a few more times to fully understand all that the author is expressing, but my question of whether or not it's a good idea to mix primary key types is answered by the author in the [comments](https://medium.com/@tomharrisonjr/over-normalize-when-i-was-learning-the-theory-and-practice-of-3rd-normal-form-the-rule-i-f3c6c8429fa9) in which he states the rule:

> "Normalize first, de-normalize only when needed."

I don't yet have a need to de-normalize. It might not matter for me yet -- not at the scale I'm working at, at least -- but I'm about to implement a model that has a polymorphic association where the foreign key might point to either a UUID or `bigint`. I *think* that will be a problem.

And then there's the question of which I should normalize to. My initial assumption about UUIDs exposed in a URL being more secure might be misplaced. ["Using a PK in a URL is generally a bad practice."](https://medium.com/@tomharrisonjr/its-a-year-or-so-after-i-have-written-the-article-but-i-think-my-recommendations-would-be-11f8f9d5a0b5)
