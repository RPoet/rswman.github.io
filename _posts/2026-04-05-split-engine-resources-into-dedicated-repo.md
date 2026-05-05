---
layout: post
title: "Split engine resources into a dedicated repository"
date: 2026-04-05 10:14:46 +0900
tags: [engine, repo, resources, workflow]
---

The resource set had grown large enough that keeping it inside the main code repository stopped being practical.

## What changed

- Removed `DomainExpansion/Engine/Resources` from the main code repository index.
- Created a dedicated resource repository for the full engine resource tree.
- Wired that resource repository to GitHub with Git LFS for large `.de` and `.fbx` payloads.

## Why it mattered

The code repo and the resource repo have different lifecycles. Scene regeneration, imported meshes, and binary payload churn should not dominate the day-to-day code history.

This split also makes future resource packaging and distribution work easier to reason about.

## Commit

- Engine commit: [`d2e1bf01`](https://github.com/RPoet/Project-Domain-Expansion/commit/d2e1bf01fe714dfcdb5dd70e6af8fd6632d02dd9)
- Resource repo: [RPoet/Domain-Expansion-Resource](https://github.com/RPoet/Domain-Expansion-Resource)

---

## Point of view TD

This is the right repository decision for the current stage of the engine.

Large asset trees and source code do not want the same workflow. They do not churn at the same cadence, they do not review the same way, and they should not impose the same clone cost on every developer action.

From a TD perspective, repository structure is pipeline structure. If resources live in the same history as code for too long, several bad things happen:

- every clone and fetch gets heavier
- review signal drops because binary churn buries code changes
- asset regeneration becomes socially expensive
- pipeline mistakes stay hidden because nobody wants to touch the big tree

Splitting the repos is not a performance optimization by itself, but it is a strong production move. It creates cleaner boundaries for ownership, tooling, and future distribution strategy.

The next bar after this split is consistency:

- resource root resolution in the engine should be explicit
- import tooling should target the resource repo intentionally
- documentation should treat the two repos as one pipeline, not two unrelated checkouts
