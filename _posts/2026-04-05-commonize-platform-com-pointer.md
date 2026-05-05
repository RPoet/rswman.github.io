---
layout: post
title: "Commonized platform COM pointer and inlined swapchain QueryInterface"
date: 2026-04-05 10:46:25 +0900
tags: [engine, platform, com, dx12]
---

This change consolidated COM smart-pointer ownership into a shared engine location instead of scattering near-identical definitions across platform headers.

## What changed

- Added a shared `PlatformComPointer.h` under the common engine layer.
- Removed duplicated COM pointer definitions from per-platform headers.
- Inlined the one-off DX12 swapchain `QueryInterface` call at its owning callsite instead of keeping a single-use helper wrapper alive.

## Why it mattered

The codebase had multiple platform-side copies of the same ownership concept. That duplication raises maintenance cost and makes it easier for platform helpers to drift.

This commit also matched an important directness rule: if only one callsite needs a platform-specific operation, keep it at that callsite instead of preserving a faux-shared helper.

## Commit

- Engine commit: [`9eb2600e`](https://github.com/RPoet/Project-Domain-Expansion/commit/9eb2600e669d2550b7408e14261bb0bf4547393e)

---

## Point of view TD

From a technical direction standpoint, this is a cleanup commit with real leverage.

The best part of it is not just code deletion. It is that the engine is choosing one ownership surface for a cross-platform concept instead of allowing platform files to each carry their own local version. COM pointer handling is not product logic, but it does sit on a dangerous seam between platform and render code. Seams like that need one clear home.

I also agree with the inlining choice for the swapchain `QueryInterface` path. Shared helpers are not automatically a virtue. If a helper exists only to hide a single platform-specific call from the only place that needs it, then the abstraction is working against readability instead of helping it.

As a TD evaluation, I would call this directionally strong because it reinforces two healthy habits:

- common ownership for truly common primitives
- direct code at the actual owning callsite when reuse is fake

That combination tends to keep low-level infrastructure from becoming a nest of thin wrappers.
