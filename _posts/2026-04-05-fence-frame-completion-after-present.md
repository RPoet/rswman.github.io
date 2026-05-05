---
layout: post
title: "Fence completion moved after present to fix resize lifetime hazards"
date: 2026-04-05 07:35:26 +0900
tags: [engine, render, dx12, stability]
---

A resize-time DX12 lifetime bug was traced back to the frame completion fence being signaled too early.

## What changed

- The frame fence signal moved to the point after `present()`.
- Resize and swapchain recreation now wait on a fence that actually covers backbuffer use through present, not just command list execution before it.

## Why it mattered

The previous ordering could release or resize swapchain resources while the GPU still had in-flight work referencing them. DX12's debug layer reported that as `OBJECT_DELETED_WHILE_STILL_IN_USE`.

This was a small code diff, but it removed a very real instability path during window resize.

## Commit

- Engine commit: [`39c94282`](https://github.com/RPoet/Project-Domain-Expansion/commit/39c94282fa5379582b10b7ce66af1eba2c97cca1)

---

## Point of view TD

This is the kind of render-side fix I want to see handled decisively: tight scope, clear root cause, and no attempt to paper over a synchronization bug with extra waiting in random places.

From a TD point of view, this commit is strong because it corrects the ownership boundary of frame completion. A frame is not complete when command lists were submitted. A frame is complete when the swapchain usage associated with that frame is no longer in flight. The fence placement should reflect that fact.

The main value here is not only bug removal. It is conceptual cleanup:

- queue submission completion is one milestone
- present-related backbuffer lifetime completion is another
- resize safety depends on the latter

That distinction tends to separate robust render backends from brittle ones.

I would treat this as a good sign for the renderer direction. The next quality step is to keep these lifecycle rules explicit in code and comments so future refactors do not regress them.
