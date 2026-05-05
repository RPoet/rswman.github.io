---
layout: post
title: "Optimized trivial vector binary reads"
date: 2026-04-05 20:46:43 +0900
tags: [engine, container, io, serialization]
---

This commit tightened the binary read path for the engine container where the payload type is trivial.

## What changed

- Optimized the engine `vector` read path for trivial binary payloads.
- Reduced unnecessary per-element handling on deserialize when the storage can be filled in one direct read.
- Kept the logic inside the engine-owned container instead of pushing callers to work around the overhead manually.

## Why it mattered

The engine is now reading a very large number of small and medium binary assets during load. In that environment, overhead in generic container deserialize paths is no longer noise.

This optimization is especially relevant after the importer and scene work exposed higher file and document counts. Small systemic wins in binary read paths start compounding quickly.

## Commit

- Engine commit: [`989e7a86`](https://github.com/RPoet/Project-Domain-Expansion/commit/989e7a8627416f4806ddcad20d539532be89030f)

---

## Point of view TD

This is the right kind of low-level optimization work: narrow, measurable in principle, and attached to a real cost center that the project is already feeling.

The engine recently moved from giant flattened assets toward many more honest scene and mesh documents. That made file count and deserialize overhead visible. In that context, optimizing trivial vector binary reads is not random micro-optimization. It is a response to a real structural shift in the content pipeline.

What makes this a good TD-facing change is that it stays in the owned container layer. That is where this optimization belongs. If callers had to special-case around slow generic reads, the architecture would be getting worse while performance got better. This commit avoids that trade.

My evaluation is positive, with one important expectation: keep this class of optimization tied to actual hot paths and measured pain. Container internals are a good place for broad wins, but they can also become a magnet for speculative cleverness if not disciplined.
