---
title: "The First Symptom"
description: "A client's calls were coming back 404, and I was already sure I knew why. I was wrong. The story of an incident I watched more senior people solve — and the two-part map of the gap between us it left me with."
pubDate: 2026-06-30
tags: ["incident-response", "debugging", "kubernetes", "caching", "akamai", "seniority", "self-taught"]
---

The client's calls were coming back 404, and I was already sure I knew why.

That sentence is the whole post, really — because the certainty in it was the problem. The previous things I've written here have been growth stories: I broke a thing, I understood it, I came out the other side a better engineer. This one is the inversion. This is the one where I was the least senior person in the room and got to watch what the gap actually looks like, in real time, under pressure.

## The symptom I fell in love with

The report was simple. A client application that fetches data from the service I work on was getting 404s, and some of the pods serving the endpoint weren't returning data.

I went straight for the explanation that fit my worldview. There was an apparent mismatch between an identifier the client was serving and the identifier the call was actually for, and I latched onto it. It was plausible. It was concrete. It was *shaped like the kind of problem I solve*.

And that, I think, is the real tell. Most of my development work lives in the data — its structure, how the system transforms it, how it gets served. I spend very little of my time in the infrastructure, or in how caching actually behaves under load. So when something broke, my instinct reached for a data-shaped cause, because that's the part of the system I can see clearly. Not an excuse. Just the honest reason my first guess pointed exactly where it did.

It was, almost certainly, unrelated.

## Someone who looked somewhere else

The shift came when the delivery manager joined the call. He didn't reach for the symptom at all. He went and read the Kubernetes logs, because he suspected something I wouldn't have thought to suspect: that the pods, while still *seemingly* running, weren't actually working — stuck in processing. Alive, but not doing the job. Green, but lying.

I want to be precise about what gave him that hypothesis, because it matters more than it first appears. It wasn't only a cooler head. He was carrying things I wasn't. He remembered a similar issue that had happened recently. And he knew the infrastructure team had made a change earlier that day — they'd been increasing the limits on the AWS machine instances. He walked into the same incident I was in holding a recent history of the system that I simply didn't have.

So while I was staring at one suspicious identifier, he was already triangulating from "what broke before" and "what changed today."

## The moves I wouldn't have made

From there the call went places my instincts never would have taken it.

They decided to scale one of the troubled clusters *down*, and then slowly scale it back up. To me, scaling down a thing that's failing to serve requests feels counterintuitive — you're removing capacity from something that's already struggling. To them it was obvious: a stuck cluster recovers faster if you let it shed its bad state and rebuild than if you keep poking it while it thrashes.

And then the conversation drifted — not randomly, but with a kind of gravity — toward Akamai. Toward the caching layer. Nobody had mentioned caching when the incident started. Watching it surface felt like watching someone who knows where the body is usually buried.

## The actual story

That's where the real signal was. Only about **24% of requests were being served from the cache.** The last time a comparable wave of traffic hit, the cache had absorbed roughly 90% of it.

Here's where I'll be honest about the edge of my own understanding, because the whole point of this post is not to overstate it: a cache hovering at 24% instead of 90% means a huge share of requests that *should* have been answered cheaply at the edge were instead falling all the way through to origin. Our applications were suddenly being asked to do many times the work they normally do for the same incoming load — and they couldn't keep up. The 404s and the stuck pods weren't the disease. They were what it looks like when the cache quietly stops protecting you and the traffic lands on the floor instead. I understand that shape now. I could not have *found* it.

## The two things I was missing

When it was over I tried to name the gap honestly, and it came out in two parts.

The first part I can practice. **I anchored on the first symptom I understood and stopped looking.** The more senior people in the room held the obvious explanation loosely — as one hypothesis among several — and kept the search open until the evidence pointed somewhere. That's a habit of mind, and habits can be trained. I can start, on the next incident, by refusing to fall in love with the first thing that fits.

The second part I can't shortcut, and that's the humbling half. They knew where to look because they *knew the system* — the infrastructure, the caching behaviour, the way pods fail without dying — and because they were carrying its recent history: the last time this happened, the change that shipped this morning. That isn't a trick. It's accumulated depth and accumulated context, and there is no version of me that has it without putting in the years.

## What it actually taught me

In my very first post here I wrote that *the senior skill isn't knowing the most tools.* I stand by it — but this is the line it was missing. The senior skill is also being hard to throw off, and knowing your own system deeply enough that the right place to look is already on the map.

I left that call slightly embarrassed and genuinely glad. Embarrassed, because I'd spent the early part of an incident confidently investigating the wrong thing. Glad, because I now have a much clearer picture of what I'm actually aiming at — and it isn't a longer list of concepts I can explain.

I can explain a cache hit ratio. I watched people who could *use* one to find a fault under pressure. Closing the distance between those two things is, I think, most of what's left of the road.
