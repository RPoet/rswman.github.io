---
layout: post
title: "Regenerated Bistro as imported entity hierarchy"
date: 2026-04-05 08:31:32 +0900
tags: [engine, bistro, import, scene, assets]
---

Once the scene import path existed, Bistro was regenerated using hierarchy-aware import instead of the old flattened mesh path.

## What changed

- Rebuilt the Bistro scene as imported entity hierarchy data.
- Generated geometry-level mesh assets under scene-specific mesh folders.
- Replaced a few giant meshes with many smaller mesh instances attached through entity hierarchy.

## Why it mattered

The headline result was not "fewer objects". It was "much smaller per-mesh section counts". The previous setup hid thousands of geometry instances inside a handful of giant mesh assets. After regeneration, the scene structure became explicit and the worst section counts dropped sharply.

That shifted the bottleneck from giant section-heavy meshes to scene and file count, which is a better problem to see clearly.

## Commit

- Engine commit: [`2810b32f`](https://github.com/RPoet/Project-Domain-Expansion/commit/2810b32f026f1f15f1ecaa4ec317ab84a9eae95a)

---

## Point of view TD

This commit is a very good example of why correct decomposition matters more than flattering top-line numbers.

A weak evaluation would say, “the scene now has thousands of entities, so it got worse.” I do not agree with that. The old state was structurally dishonest. It hid scene complexity inside oversized mesh assets and then made the renderer pay for that distortion in ugly ways.

The new state is better because:

- geometry granularity is explicit
- scene ownership is explicit
- section counts are attached to the right unit
- future systems can now optimize the real problem instead of a flattened artifact

That said, this commit also exposes the next bottleneck with painful clarity: file count and load orchestration. That is not a failure of the direction. It is the natural next issue after import semantics are corrected.

From a TD lens, this is exactly the kind of trade I want:

- first, make the data model truthful
- then, optimize the truthful model

The bad alternative would have been to keep optimizing a fundamentally misleading representation.
