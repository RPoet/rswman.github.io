---
layout: post
title: "Marked the temporary transform path with TEMP_ and TODOs"
date: 2026-04-05 10:16:56 +0900
tags: [engine, transform, temp, cleanup]
---

The transform work done during the import experiments was intentionally not final, and the code needed to say that clearly.

## What changed

- Added `TEMP_` prefixes to the temporary transform helpers and buffers introduced during the experiment.
- Added explicit TODO markers around the temporary world-transform query path.
- Left the path buildable and visible instead of silently baking experimental behavior into normal names.

## Why it mattered

There was already a clearer direction for the final design: cached world transform on entities, subtree updates in world update, and runtime sync after transform resolution. Until that design lands, the experimental path should be obvious in code review and in future cleanup passes.

This commit was less about functionality and more about making technical debt explicit instead of ambiguous.

## Commit

- Engine commit: [`88c2f7e1`](https://github.com/RPoet/Project-Domain-Expansion/commit/88c2f7e1cede1239f5d57a829759d13000f7f8f7)

---

## Point of view TD

I rate this kind of commit highly when it is used deliberately.

Temporary architecture is dangerous mostly when it looks permanent. Naming matters. Marking experimental transform code as temporary and surrounding it with TODOs is not cosmetic. It is a form of technical honesty.

From a TD perspective, this commit is good because it does three useful things:

- it stops accidental normalization of an experimental path
- it lowers the chance of future systems depending on the wrong abstraction
- it preserves working context without pretending the design is settled

The real value here is organizational. Once a codebase starts absorbing temporary transform queries into normal naming, teams unconsciously begin building around them. That is how long-term architectural debt sneaks in.

The correct next step remains clear:

- entity-owned cached world transform
- dirty subtree propagation in world update
- render and camera sync after transform resolution

Until that lands, the code should continue to advertise that the current path is transitional.
