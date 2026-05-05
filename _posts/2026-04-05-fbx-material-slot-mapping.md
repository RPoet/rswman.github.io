---
layout: post
title: "FBX material slots mapped into mesh sections"
date: 2026-04-05 07:30:00 +0900
tags: [engine, fbx, materials, mesh]
---

This change fixed the boundary between imported FBX material assignment data and the engine's mesh section model.

## What changed

- FBX import now carries material slot information into generated mesh sections instead of assuming section index and material index are the same thing.
- Mesh runtime lookup follows `section -> material slot -> material asset` rather than binding directly by section ordinal.
- The importer started reading more of the FBX-side material assignment data so mesh data and runtime binding rules are aligned.

## Why it mattered

The old path flattened too much information. That made Bistro-style meshes hard to reason about, because section count exploded while material intent was being lost on the way into engine data.

This commit did not solve the full scene import problem yet, but it put the mesh data model on the right side of that boundary.

## Commit

- Engine commit: [`4704659b`](https://github.com/RPoet/Project-Domain-Expansion/commit/4704659b6b47f288aa927849865515839d684c8f)

---

## Point of view TD

From a technical direction standpoint, this is the first commit in the batch that moves the importer away from accidental behavior and toward an explicit asset contract. That matters more than the immediate runtime effect.

The strongest part of this change is the separation of concerns:

- mesh topology and section layout belong to `MeshAsset`
- material binding choice belongs to the instance side
- FBX polygon material assignment should map to slot semantics, not section ordinal coincidence

That is the right direction for any engine that wants to scale past toy assets. Without this split, every later effort around material override, instancing, or scene import ends up fighting a bad assumption baked into the mesh format.

What I like less is that the engine still relies on section granularity as the render submission unit. This commit improves correctness, but it does not yet improve the structural cost model. In other words, it makes the data less wrong, not yet cheap.

As a TD evaluation, I would rate this change as strategically correct and foundational. It does not finish the problem, but it removes one of the false equivalences that would otherwise poison every later import and rendering decision.

The next technical bar after this point should be:

1. Keep `section -> material slot` as the durable contract.
2. Keep mesh asset reuse independent from instance material override.
3. Refuse to let scene reconstruction flatten back into section-index binding by accident.
