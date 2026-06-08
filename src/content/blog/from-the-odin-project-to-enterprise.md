---
title: "From The Odin Project to 100+ Microservices"
description: "What no tutorial taught me about enterprise software — told through the first real thing I owned: an end-to-end CI pipeline I built by breaking it, over and over."
pubDate: 2026-06-08
tags: ["self-taught", "career", "ci-cd", "jenkins", "kubernetes", "testing"]
draft: false
---

I finished The Odin Project, and then I opened the codebase.

That's the whole story, really. But it deserves more than one line, because the gap between those two moments is where I actually became an engineer.

## Where I started

I'm a self-taught developer. No computer science degree, no bootcamp cohort, no internship pipeline. Back in 2021 I quit my day job and spent about ten months learning to code full time — mostly through [The Odin Project](https://www.theodinproject.com/) — before I sent out my first applications. TOP is genuinely excellent. It taught me to build full applications from an empty folder — routing, databases, auth, tests, deployment. By the end I could ship a working app on my own, and that confidence is what got me in the door.

What it did not prepare me for was the door itself.

Because the skill TOP teaches is *building small things from scratch*. The skill the job demands is *safely changing enormous things you didn't build and can't fully understand*. Those are not the same skill. They are barely related.

## The assignment that scared me

A few months into my first proper engineering role, I was handed ownership of an end-to-end testing pipeline for a group of four backend services. These services were a self-contained corner of a much larger platform — over a hundred microservices handling content ingestion, processing, and distribution. My job: make it so that, on every change, we could spin the whole corner up, run a suite of behavioural tests against it, and tell the team whether anything broke.

I said yes immediately. Then I went and quietly panicked, because I had never used Jenkins, had never written a line of Kubernetes config, and could not have confidently told you what "CI" actually stood for in practice versus theory.

My very first commit on this project tells you everything. It added a new module to the deployment pipeline, wired up a build definition, and created a brand-new Kubernetes namespace with a resource quota and a config map. Every single noun in that sentence was new to me. I was copying patterns from neighbouring services, reading the output, and praying.

## The invisible 80%

Here is the thing tutorials structurally cannot show you. A tutorial is a greenfield project with a known-good ending. The author already knows the answer, so the path is clean. Real work has no author and no known answer. And most of it is the part that never appears in a tutorial at all.

Nobody had walked me through:

- **CI orchestration** — how a pipeline is defined, staged, and triggered, and what happens when one stage lies to you about success.
- **Containerisation** — writing Dockerfiles for services *and* for the test runner itself, so the whole thing is reproducible.
- **Kubernetes** — namespaces, resource quotas, network policies, secrets, pod specs. An entire vocabulary of "where does the code actually run."
- **Test infrastructure** — standing up a throwaway database, mocking an external dependency the services expected to call, and wiring four services plus a message broker into one coherent, ephemeral environment.

None of that is in a curriculum. It's the 80% of the job that's invisible until you're standing in it.

## Learning by breaking things

You cannot unit-test a deployment pipeline. There's no local "run" button that proves it works. The only way to know if a pipeline is correct is to run it, in the real environment, and watch what it does. So that's how I learned: by breaking it, in public, repeatedly.

If you read my commit history from that period, you can watch me learn in real time. "Fix the typo in the report filename." "Fix the environment name in the quota config." "Increase the test timeouts so they can actually finish in the deployed environment." "Shorten the service names." "Remove the debugging code I left in." Each of those is a small humiliation that was, at the time, a genuine mystery costing me an afternoon.

I built a small piece of tooling to parse the test reports and post a clean summary to a team chat channel — pass/fail counts, a link to the full report. The first version was crude: a few classes hammered together to get *something* on the screen. It worked. That mattered more than it being pretty.

## The lesson no tutorial gave me

Then came the subtle one — the moment I think I actually started thinking like an engineer rather than a person who writes code.

For weeks, the pipeline was green. Beautifully, reliably green. Until someone pointed out that a test had clearly failed, and the build had passed anyway. The test runner was exiting successfully regardless of the results. I had built a smoke detector that was wired to never go off.

The fix was a handful of lines: capture the test outcome, and make the build exit with a failing status if anything failed. Trivial code. But the realisation behind it was not trivial at all: **a green pipeline that cannot turn red is worse than no pipeline, because it manufactures false confidence.** No tutorial teaches you that, because no tutorial has ever lied to you about whether your tests passed.

Around the same time I learned to tear down old deployments before standing up new ones — that resources don't clean themselves up, and that "it worked on the last run" is not the same as "it works." Operational reality, learned by hitting the wall.

## Make it work, then make it right

The part I'm proudest of came nearly a year later. I went back to that crude little reporting tool — the one I'd hammered together just to get something working — and I refactored it properly. I introduced real model types for the test results, separated the parsing from the formatting, and gave it the structure it always deserved.

What struck me wasn't the refactor. It was that I could *see* the difference between the engineer who wrote the first version and the one doing the rewrite. A year earlier I'd been copying patterns and praying. Now I was making deliberate design decisions and could explain every one of them. The growth wasn't abstract. It was sitting right there in the diff.

"Make it work, then make it right" is advice you read everywhere. Living through both halves of it on your own code, a year apart, is how it stops being a slogan.

## What it actually taught me

The deepest lesson had nothing to do with Jenkins or Kubernetes specifically. It was this: **enterprise code is communication.** That pipeline existed so that other people — people who would never read its internals — could trust a green checkmark and a chat message. Its entire value was in clearly telling humans something true about the system. The tooling was incidental. The communication was the point.

That reframed everything for me. The senior skill isn't knowing the most tools. It's writing things — code, tests, infrastructure, documentation — that other people can rely on without having to ask you. Reading existing systems before changing them. Being honest about uncertainty. Leaving the campsite cleaner than you found it.

The Odin Project got me to the door. Breaking that pipeline, over and over, until I understood why it broke — that's what got me through it.

I've since been applying the exact same "learn in public, break it until you understand it" habit to async Rust and Bitcoin tooling on my own time.
