---
title: "The Same Pipeline, a Year Later"
description: "Last year I built an end-to-end test pipeline by breaking it. This year I deleted half of it. The story of moving a multi-service E2E suite off Kubernetes and onto Testcontainers — and what the diff says about who I've become."
pubDate: 2026-06-15
tags: ["testing", "testcontainers", "kubernetes", "kafka", "ci-cd", "java"]
draft: false
---

[Last time](/blog/from-the-odin-project-to-enterprise/) I wrote about the first real thing I owned as an engineer: an end-to-end test pipeline for a corner of a large content platform, which I learned to operate by breaking it in public, over and over.

This post is the sequel. Because a year later I went back and deleted a big chunk of it — on purpose — and replaced it with something better.

## How the pipeline used to work

The old E2E suite ran on Kubernetes, and the shape of it was completely normal for an enterprise platform. To exercise four-plus services together, the pipeline would:

- build a fat `e2e-tests.jar` containing the whole Cucumber suite,
- bake that jar into a Docker image with its own `Dockerfile`,
- schedule a **pod** into a Kubernetes namespace using a `pod.yml` spec — `securityContext`, resource limits, a `configMap` mounted for config, and the Slack token pulled from a `secretKeyRef`,
- run a couple of shell scripts (`runTests.sh`, `start_e2e_tests.sh`) that set the JVM and GC flags and finally launched the jar,
- and scrape the test reports back out afterwards.

It worked. It had been working for a year. But read that list again and notice what's missing: **a way to run the whole thing on your own machine.** The tests were real, but the environment they needed was a remote Kubernetes namespace. Locally, the best you could do was run the Gradle task against a profile and hope it resembled the deployed reality.

## The friction that finally got to me

The cost of that design wasn't dramatic. It was a tax. Every time a test failed in CI, the loop to investigate was: change something, push, wait for an image to build, wait for a pod to schedule, wait for the services to come up, wait for the suite, read the logs, repeat. The feedback cycle was measured in coffees, not seconds.

And there's a subtler problem with "you can only run it in the cluster." It quietly concentrates knowledge. The pipeline becomes a thing only a few people can poke at, because poking at it requires the whole Kubernetes apparatus. That's the exact opposite of what I'd decided test infrastructure is *for* — it's supposed to let other people trust a result without having to ask me.

## The rebuild: Testcontainers

So the migration (one ticket, ~1,500 lines changed across 35 files) tore out the Kubernetes scaffolding — the `pod.yml`, the `Dockerfile`, the launch scripts, the custom Cucumber runner — and stood the whole environment up *from inside the test process itself* using [Testcontainers](https://testcontainers.com/).

The new design has two halves:

- **Infrastructure containers** — PostgreSQL and Kafka (via Testcontainers' `KafkaContainer` on `apache/kafka:3.7.0`), wrapped in small `Startable` types so the test harness controls their lifecycle.
- **Service containers** — the actual services under test (proxy-mock, consumer, atomiser, curator, stitchup), each declared with `dependsOn()` so it waits for the infrastructure before it boots.

Startup is parallel where it can be (`Startables.deepStart(...)`), ordered where it must be (`dependsOn(...)`), and the containers come up *before* the Spring context so the test app connects to real, running dependencies. A Cucumber `@AfterAll` hook tears everything down again. The entire ephemeral world — a message broker, a database, and five services wired together — is now described in Java, next to the tests that use it.

## The details that make it real

The clean version of this story is "I swapped Kubernetes for Testcontainers." The honest version is that most of the work was in the seams.

The biggest one: **the same code has to run in two very different places.** Locally, containers talk to each other over a Testcontainers bridge network with exposed ports. In CI — running under Podman — that doesn't work the same way, so the harness flips into a host-network mode and waits on log messages ("database system is ready to accept connections", "Started ... in ...") instead of mapped ports. One environment, abstracted behind a flag, so the suite behaves identically whether it's on my laptop or in the pipeline.

There's also a `SKIP_TESTCONTAINERS` escape hatch for when you want to point the suite at something already running, and — the part I care about most — a `Makefile` target so that running the full multi-service E2E suite locally is now one command. The thing only a few people could touch is now the thing anyone can touch.

## Closing the loop on last year's lesson

Last year's hardest lesson was that **a green pipeline that cannot turn red is worse than no pipeline**, because it manufactures false confidence. I'd shipped a test runner that exited successfully no matter what the tests did.

This time, part of the same effort went into the reporting tool from that first post — the one that posts pass/fail summaries to Slack. I moved the notification out of the test runner and up to the GitHub Actions workflow level, specifically so it still fires when something breaks *before the tests even run*. A build that dies during environment setup used to go silent. Now it still tells a human the truth. Same lesson as last year — only this time I designed for it up front instead of discovering it the hard way.

## What the diff says about a year

Last year's commit history was a trail of small humiliations: fix the typo in the report filename, fix the environment name in the quota config, increase the timeouts. Copying patterns from neighbouring services and praying.

This year's diff is different in character. It deletes more than it adds in places. It introduces abstractions — `Startable`, container factories, a network-mode switch — because I could see the shape of the problem before writing the code, not after. A year ago I was learning what a Kubernetes namespace even was. This year I decided we didn't need one for this job, and I had the judgement to back that up.

"Make it work, then make it right" was the slogan last time. This was the *make it right* — except the thing I was making right was a whole environment, not a single tool, and the growth wasn't abstract. It was sitting right there in the diff again.

The Odin Project got me to the door. The pipeline taught me to be an engineer. And apparently the same pipeline is still teaching me — I just get to choose the lessons now.
