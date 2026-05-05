---
layout: post
title: "Reimported mesh assets for section material slots"
date: 2026-04-05 07:30:23 +0900
tags: [engine, assets, import, mesh]
---

After changing the mesh section format, the generated mesh assets had to be rebuilt so authored data matched the new runtime expectations.

## What changed

- Reimported the Bistro meshes and the basic sample meshes.
- Regenerated `.de` and `.deasset` mesh outputs to match the section-to-material-slot mapping introduced in the previous commit.

## Why it mattered

The engine-side code and the generated assets have to agree on the serialization layout. During this stage, backward compatibility was intentionally not preserved, so the correct move was to reset and regenerate.

This was still part of the mesh-only import path. Scene hierarchy import came later.

## Commit

- Engine commit: [`137538bc`](https://github.com/RPoet/Project-Domain-Expansion/commit/137538bc4d85d52dcac219c8dae3ebd1da1e8637)

---

## Point of view TD

This is an execution hygiene commit, and engines often fail here by pretending schema changes are harmless. They are not.

From a TD perspective, the choice to regenerate instead of layering temporary backward compatibility was correct for the project phase you are in. Early engine development benefits from aggressive reset-and-rebuild when the format contract is still in motion. Carrying legacy baggage too early slows iteration and hides structural mistakes behind compatibility code.

The important evaluation point is not the asset diff itself. The important part is that the team accepted this rule:

- data schema changed
- compatibility was intentionally not promised
- source assets were reimported immediately

That is a healthy discipline for a pre-release engine.

The caution is operational rather than architectural. Once the project grows, mass reimport cost becomes a pipeline problem of its own. When that happens, you will need:

- deterministic import runners
- import manifests
- asset provenance tracking
- import result validation

So this commit is good, but it also marks the moment where pipeline tooling becomes mandatory if import churn continues.
