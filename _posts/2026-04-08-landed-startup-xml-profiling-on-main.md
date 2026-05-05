---
layout: post
title: "Landed startup XML profiling on main"
date: 2026-04-08 06:27:36 +0900
tags: [engine, startup, profiling, xml, perfetto]
---

This commit put real startup-load measurement on `main` instead of leaving initial load investigation at the level of guesswork and anecdotes.

## What changed

- Added application/session-side startup capture flow and command handling for profiling runs.
- Added profiler ownership surfaces, including a null backend and a Perfetto backend.
- Routed startup-relevant paths such as world loading and XML document work through measurable scopes.
- Kept path and loader concerns on the engine side instead of turning the entrypoint into a startup-logic blob.

## Why it mattered

At this point the imported Bistro startup path was already slow enough that "it feels heavy" was not a useful diagnosis anymore.

The engine needed a repeatable capture path so later optimization work could answer concrete questions:

- how much time is in startup overall
- how much of that sits in `Framework::loadWorld`
- how much is XML read/parse cost
- whether the real problem is parser cost, file-count fan-out, or both

The first baseline captured through this path made the bottleneck much clearer: `startup_capture=6114.956 ms`, `Framework::loadWorld=5863.383 ms`, `XML total read+parse=2400.080 ms`, `documents=11749`.

## Commit

- Engine commit: [`f0ba413a`](https://github.com/RPoet/Project-Domain-Expansion/commit/f0ba413ac9b8ba74fb223d4e2440230d831bd29e)

---

## Point of view TD

From a TD perspective, this is exactly the kind of infrastructure commit that should land before trying to "optimize startup" in a more heroic way.

The main value is not that profiling code exists. The value is that the engine now has a shared measurement path on `main` that later parser, manifest, and scene-pack work can compare against. That changes startup optimization from taste-driven cleanup into evidence-driven iteration.

I also like the ownership direction here. The patch introduces profiler/backend plumbing and session-side capture flow, but it does not pretend that `Main.cpp` should personally own every startup rule. That matters, because performance work often becomes messy when measurement code is allowed to dissolve architecture on the way in.

The next expectation after a commit like this is discipline: every startup claim after this point should come with the same-path before/after numbers.
