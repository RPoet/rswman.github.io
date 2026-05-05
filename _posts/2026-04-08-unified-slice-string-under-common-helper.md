---
layout: post
title: "Unified scattered slice-string handling under a common helper"
date: 2026-04-08 16:54:19 +0900
tags: [engine, common, strings, xml, parsing]
---

This commit cleaned up a small but spreading pattern: multiple subsystems had started carrying their own local slice-string behavior even though the underlying operation was generic.

## What changed

- Added `Engine/Common/StringSlice.h` as the shared owner for the reusable slice-string behavior.
- Replaced local or duplicated substring logic in `ApplicationRunOptions`, `EditorCommandReplay`, `XML`, `DiskLoaderModule`, `FbxBinaryParser`, `ObjMeshParser`, and `ShaderPackageModule`.
- Kept the generic text operation in `Common` instead of letting each consumer grow its own slightly different version.

## Why it mattered

This is a small refactor, but it fixes the right kind of problem early.

Once a generic text helper starts appearing in XML parsing, replay, disk loading, mesh parsing, and shader package handling, it is no longer a local implementation detail. Leaving those copies in place means future behavior changes get paid multiple times, and the copies usually drift.

Centralizing the helper now keeps future parser and loader cleanup moving in one direction instead of multiplying local convenience code.

## Commit

- Engine commit: [`b014df6c`](https://github.com/RPoet/Project-Domain-Expansion/commit/b014df6c6e65bad723487a509d6779024d6965e5)

---

## Point of view TD

From a technical direction standpoint, this is the right kind of commonization.

The change is not "make a helper because helpers are nice." The change is "this operation is already spreading across real owners, so give it one real home before the spread hardens into maintenance debt."

That distinction matters. Fake commonization creates wrappers with one callsite. Good commonization deletes repeating policy from multiple files and moves it to the shared layer that should own it.

This commit also pairs well with the current XML/startup work. If the parser and loader are going to be tuned further, shared string-slice behavior should already be centralized so those changes do not have to chase multiple consumer-local copies.
