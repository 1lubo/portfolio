---
title: "The Best Hours Go to the Job"
description: "How do you combine a full-time Java job with learning Rust and building things on the side? The honest answer is that you don't balance it — the job takes your best hours, the side work crawls, and the only reason any of it survives is that it's all the same habit."
pubDate: 2026-07-07
tags: ["rust", "java", "learning-in-public", "self-taught", "career", "side-projects"]
draft: false
---

Here's a question I keep circling myself, even though nobody's ever actually put it to me: how do you hold down a full-time job in a Java codebase and still learn Rust and build your own things on the side? What's the system?

There is no system. And I think pretending there is one is exactly where this goes wrong.

So this post is the honest version. Not a routine to copy, not a time-blocking template — just what combining these three things actually looks like when you're a self-taught engineer with a mortgage-shaped need to keep the day job going well.

## The premise I had to give up

The word people reach for is *balance*, and it's the first thing I had to throw out.

Balance implies a scale that can sit level — that on a good week the job and the learning and the side projects each get their fair share, and the trick is finding the equilibrium. That was never true for me, and chasing it just made me feel like I was failing at all three simultaneously.

Here's the part nobody frames as a strategy: **the best hours go to the job, and they should.** The job is where I'm at my sharpest — rested, caffeinated, morning brain, a full day of uninterrupted focus. That's not a compromise I resent. It's the correct allocation. The work is real, people depend on it, and it's also where I've done the most growing as an engineer. Giving it the top of my energy isn't the thing standing between me and my side projects. It's the whole point.

Which leaves the side work with what's left. And what's left is not much.

## What "on the side" actually looks like

If I'm honest about my rhythm for the personal stuff, the truest thing I can say is: **scattered and inconsistent, whenever energy allows.** I want to sit with that instead of dressing it up, because the dressing-up is the lie.

It is not every evening. It is not disciplined weekend blocks. It's a tired hour after dinner where I get one function to compile. It's a Saturday morning that was going to be productive until it wasn't. It's a stretch of nothing at all when a hard sprint at work empties the tank completely, followed by a burst when something reignites.

The direct consequence, stated plainly: **the side projects move at a crawl.** They progress at a fraction of the speed they would if they were my full-time focus, because they are structurally never getting my full-time focus. For a while I read that slowness as a verdict on my discipline. It mostly isn't. It's just arithmetic. The best hours are already spoken for.

## The first way I got it wrong

My first real attempt at Rust died of exactly this misunderstanding.

I [picked a Tor proxy](/blog/what-spring-was-doing-for-me/) — networking and crypto and async all at once — and tried to treat the side project like a second job with its own ambitions. Learn the hardest thing, in the scarcest hours, while tired. Every line introduced three new problems at once, and with an hour of depleted evening attention I couldn't make a dent. So I bounced off it, and told myself it was a discipline problem.

It wasn't. It was a design problem. I'd scoped my side work as if it would get the resources of a main job, when by definition it never could. A project that needs your best hours to make any progress is a project you will not finish on your scraps.

## The reframe that actually works

Two things changed, and together they're the only real answer I have.

The first: **stop making the side work compete with the job, and start letting the job feed it.** When I came back to Rust, I didn't pick something new and heroic. I took a thin slice of a service I actually maintain at work — a boring in-memory CRUD API I understood cold — and [rebuilt it in Rust](/blog/what-spring-was-doing-for-me/). Suddenly the only unknown was the language, and a tired evening hour was enough, because the domain came for free from the day job. The Java work stopped being the thing stealing time from Rust and became the thing *funding* it. And it ran the other way too: understanding what Rust made me spell out by hand taught me what Spring had been silently doing in my Java all along. The two codebases quietly pay into each other.

The second: **treat the projects as learning-driven, not product-driven.** What I'm building on the side isn't one polished product I'm ashamed of not shipping. It's a mix of small things whose actual output isn't the thing — it's the engineer who understood something by building it. Once the goal is *understanding* rather than *shipping*, "slow" and "half-finished" stop being failures. A crawl is fine when the point of the walk was the walk.

## The one habit underneath all three

Strip away the specifics and there's a single thread running through the day job, the Rust, and the side projects — and it's the same one that shows up in everything I've written here.

I [learned my first pipeline by breaking it in public, over and over](/blog/from-the-odin-project-to-enterprise/), until I understood why it broke. I learned Rust by rebuilding something I knew and letting the compiler break me until I'd genuinely decided what I meant. The side projects are the same move again: pick something slightly beyond me, do it in the open, break it until it makes sense.

That's what actually combines these three things. Not a schedule. Not balance. It's that they're not really three separate pursuits fighting over one calendar — they're one habit, *learn in public and break it until you understand it*, pointed at whatever's in front of me. The job gets the best hours. The side work gets the scraps and moves at a crawl. And it all still compounds, because underneath it's the same person doing the same thing, just in different codebases.

So if you're waiting for the week where it finally balances before you start the thing on the side — don't. It won't balance. Give the job your best hours, give the side project your honest scraps, make sure the two feed each other, and let it be slow. Slow and compounding beats fast and imaginary every time.
