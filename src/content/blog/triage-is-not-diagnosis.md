---
title: "Triage Is Not Diagnosis"
description: "A support week is mostly filter work — until one recurring post-deploy queue issue earns the expensive kind of thinking. On triage versus diagnosis, and investigating a ghost previous fixes never killed."
pubDate: 2026-08-10
tags: ["incident-response", "support", "debugging", "databases", "on-call", "seniority"]
draft: true
---

Most of a support week is not drama. It is sorting.

You learn quickly which pings are real pressure and which are noise wearing an incident costume. You decide, often in minutes, how much thinking a ticket is allowed to cost — because if every thread gets a full investigation, you will burn the week and still miss the one that mattered.

That is the part feature work almost never trains. On a feature ticket you choose the depth. You can stay in the expensive mode for days on a problem you picked. A support week makes depth a budget.

## The cheap tickets

The early part of the week was the usual mix. A few self-inflicted ones — something we shipped, or something adjacent to a change we owned — that you take, fix or roll back, and close without writing a novel. A few aimed at the wrong team entirely, where the useful move is a clean handoff, not a heroic dig into someone else's system. None of those needed a theory of the platform. They needed a filter with good taste and a short half-life.

I am leaving them there on purpose. They matter only as proof that the week had volume. The real story is the ticket that refused to die in triage.

## When a ticket earns the depth

After a deploy — and depending on the environment, sometimes only if someone is watching closely enough to notice before it clears — an event-queue cleanup task starts a heavy query against the database, gets stuck, and the DB begins to overheat. Other event-related work slows. Services take too long to become fully operational. The system does not fall over in a cinematic way. It just comes up wrong: delayed, congested, expensive.

On a quiet afternoon you could almost triage that as "DB busy after deploy, give it time." That answer is available. It is also how this class of issue survives.

Three things upgraded it out of the cheap pile.

First, it is **recurring**. It is not a one-off spike; it rides in on deploys like a bad habit.

Second, it is **shaped by deploy timing**. The fact that it shows up in the wake of a release is itself a clue — warm state, backlog drain, cleanup catching volume an idle system never hits, competing tasks starting together. "After deploy" is not colour. It is part of the symptom.

Third, people have **already tried to fix it**. That is the detail that changes the job. You are not walking into a green field with a plausible patch. You are walking into a ghost that partial fixes have failed to kill. So "find something that looks related and change it" is not investigation. It is how you add another entry to the pile of attempts that did not stick.

Triage is the decision that this combination is allowed to cost real thinking.

## What deep mode looks like when resolution is not guaranteed

I want to be honest about what "investigating" meant here, because it was not a straight line to a root cause and a merged PR. Previous attempts had already treated pieces of this — the stuck query, the cleanup behaviour, some corner of the schedule or the load — and the ghost came back. That history is data. It suggests the recurrence does not live in any one of those pieces alone. It lives in the interaction: cleanup work meeting deploy-time load meeting a database that is suddenly asked to do too much at once, while the rest of the event machinery waits downstream.

So the useful moves were less glamorous than a breakthrough.

**Separate the symptom layers.** Slow services and delayed tasks are what people feel. Database pressure is what the dashboards shout. The cleanup query is the concrete thing that gets stuck. Those are not the same layer. Falling in love with the first red panel is how you re-enact the partial fix.

**Take "after deploy" seriously as a constraint.** Whatever is true about this failure has to still be true in the window when a release has just landed — when work that was quiet starts moving, when cleanup may be facing a different shape of backlog than it sees at steady state, when several kinds of startup and catch-up collide. A theory that only makes sense on a calm Tuesday is the wrong species of theory.

**Read the failed past without contempt.** Earlier attempts were not necessarily stupid. They were incomplete. The question is not "why did nobody fix this?" It is "what did each change assume, and what did it leave untouched?" That is a slower question. It is also the only one that respects the fact that simple answers have already been tried.

**Hold the ending loosely.** The product of this stretch of thinking is not obliged to be a fix in the same breath as the incident thread. Sometimes the honest output is a sharper model: a better account of why this earns depth, a tighter set of questions for the next attempt, a refusal to ship another patch that only silences the dashboard for one environment.

I do not have the closed root-cause story yet. I am not going to invent one for the cadence of a blog post. What I have is a clearer picture of the kind of problem this is — and a clearer picture of why the cheap close would have been a way of looking away.

## What the week actually trained

Feature work rewards staying deep. You pick a problem, you live in it, you come out with code and a story. Support work rewards knowing when *not* to go deep — and then recognising the rare ticket where cheap answers are how the bug keeps its lease.

The light tickets trained the filter. The queue issue spent the depth the filter had saved. That rationing is the skill. Deep diagnosis on every ping is not thoroughness; it is how you arrive at Friday with no attention left for the ghost that has been surviving partial fixes for longer than anyone wants to admit.

Triage is not diagnosis. Confusing them is how recurring issues stay recurring. Keeping them distinct is how you still have enough mind left, mid-week, to investigate the one that earned it — even when the honest end of that investigation is not a victory lap, but a better next question.
