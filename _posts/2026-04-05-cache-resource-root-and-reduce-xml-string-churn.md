---
layout: post
title: "Cached resource root and reduced XML parser string churn"
date: 2026-04-05 20:47:14 +0900
tags: [engine, loading, xml, io, resources]
---

This commit targeted the startup and load path more directly by removing repeated resource-root resolution work and cutting some XML-side string churn.

## What changed

- Cached the resolved resource root instead of re-searching for it repeatedly.
- Tightened XML parser string handling to reduce avoidable temporary churn during document processing.
- Kept the changes inside the loader and XML ownership layers instead of leaking workaround state outward.

## Why it mattered

Once the resource tree moved into a separate repository and the imported Bistro scene started loading many more files, infrastructure cost that used to be hidden became visible.

Repeated path discovery and excess string churn are exactly the kinds of problems that hurt load latency without showing up as obvious feature bugs. Fixing them early is worthwhile because they multiply across every document load.

## Commit

- Engine commit: [`c94b6477`](https://github.com/RPoet/Project-Domain-Expansion/commit/c94b6477196826b4150fd15c4973417decd4052d)

---

## Point of view TD

From a TD perspective, this is a good example of the engine reacting to newly exposed bottlenecks in the right order.

After the scene-import corrections, the project now pays more honest load-path costs. That naturally moves attention toward loader and parser overhead. Caching the resource root and reducing XML string churn is a sane first wave because those costs hit every file, not just one specific asset class.

I like this direction because it improves infrastructure without distorting ownership:

- path resolution still belongs to the disk-loading side
- XML string behavior still belongs to the XML layer
- callers do not need to carry extra state just to avoid repeated internal work

That is the hallmark of a good engine optimization. The system gets cheaper, but the rest of the engine does not get more complicated.

The next TD expectation after this point is measurement discipline. Once load-path optimization starts, every new improvement should be tied to a concrete cost breakdown so the engine does not drift into "cleanup that sounds fast" instead of changes that are actually moving startup and scene-load time.
