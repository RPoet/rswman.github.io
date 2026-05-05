---
layout: post
title: "Added FBX entity hierarchy scene import"
date: 2026-04-05 08:29:39 +0900
tags: [engine, fbx, import, scene]
---

This was the point where the importer stopped pretending that every FBX file should become one flattened mesh asset.

## What changed

- Added a scene-style import path that uses FBX model and connection data to build entity hierarchy information.
- Kept the split between `Geometry -> MeshAsset` and `Model -> Entity`.
- Extended the mesh parser surface so FBX can be brought in as hierarchy data instead of always being baked into one big mesh blob.

## Why it mattered

The earlier mesh-only path was the direct cause of the Bistro section explosion problem. FBX geometry instances and scene nodes were being flattened together, which destroyed the original scene structure and forced huge monolithic assets.

This commit established the import boundary needed for proper scene reconstruction.

## Commit

- Engine commit: [`05434b98`](https://github.com/RPoet/Project-Domain-Expansion/commit/05434b988d0ecf3bde625298c5269638d1b7994c)

---

## Point of view TD

Architecturally, this is the most important commit in the set.

Why? Because it fixes the import worldview, not just a symptom. Engines become expensive to maintain when they confuse scene description with render payload. The correct mental model is:

- scene graph describes placement and relationship
- mesh asset describes reusable geometry payload
- material asset describes shading intent
- instance-side components connect those pieces

Once that split is respected, many later systems become straightforward:

- scene authoring
- instancing
- hierarchical culling
- prefab-style reuse
- import/export round-tripping

From a TD perspective, this commit moves the engine from “custom importer that happens to load files” toward “content pipeline with a valid intermediate structure.”

The remaining warning is that hierarchy import by itself is not enough. If the runtime transform model and the asset loading model remain naive, the engine will simply trade one bottleneck for another. But that is an acceptable next problem. This commit earns that next problem by finally exposing the scene structure honestly.
