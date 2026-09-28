---
title: Review: rollback-and-per-volume-locking
type: scratch
status: draft
author: reviewer
tags: []
links: []
created: 2026-09-28
updated: 2026-09-28
---

# Review: rollback-and-per-volume-locking

## Verdict: NeedsChanges

## Issues

### [CRITICAL] The stated prerequisite, restructure-for-testability-and-add-tests, is not present in the repository: src/app.ts and src/*.test.ts do not exist, createApp does not exist, Rbd has no injectable command runner, and package.json has no test script. The handlers are still in src/server.ts, which this goal forbids changing. Make the prerequisite an explicit dependency and dispatch this goal only after it lands, or revise the permitted files and scope.

### [MINOR] Clarify Create rollback when rbd.map throws after creating a mapping: how should the worker determine whether this request owns the mapping before calling unMap? Rollback must not unmap a pre-existing mapping.

### [MINOR] Mount-directory cleanup should remove only a directory created by the failed request; Rbd.mount currently calls recursive mkdir without recording whether the directory existed beforehand.

## Verified
- ❌ src/app.ts does not exist
- ❌ createApp does not exist
- ✅ src/rbd.ts and Rbd exists
- ❌ Rbd injectable command runner does not exist
- ✅ Rbd.isMapped/map/unMap/create/makeFilesystem/remove/mount/unmount exists
- ❌ src/*.test.ts does not exist
- ❌ package.json test script does not exist
- ✅ src/server.ts route handlers and mountPointTable exists
- ✅ src/mountPointEntry.ts and references exists
- ✅ package.json build script exists

## Recommendation
Add an explicit dependency on the testability restructure and verify it has landed before dispatch. Once it has, this is a focused, feasible goal; clarify mapping ownership and directory cleanup while retaining the proposed tests.
