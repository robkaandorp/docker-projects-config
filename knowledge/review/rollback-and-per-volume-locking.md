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

### [MAJOR] Create’s final unMap call can fail after create, map, and makeFilesystem succeed. The goal does not specify whether to retry cleanup, preserve the formatted image, or leave the mapping in place. Define this failure case explicitly and add a test; removing a successfully formatted image without a clear rule risks data loss.

### [MINOR] Clarify that Rbd.mount must preserve its existing command-specific error message when rethrowing after directory cleanup. Its current tests expect a wrapped mount error, not the raw command-runner error.

## Verified
- ✅ src/app.ts and createApp exists
- ✅ src/app.ts Create, Remove, Mount, and Unmount handlers exists
- ✅ src/rbd.ts Rbd and injectable command runner/filesystem exists
- ❌ src/rbd.ts FileSystem.existsSync does not exist
- ✅ src/rbd.ts Rbd create, map, makeFilesystem, unMap, isMapped, mount, unmount, and remove exists
- ✅ src/app.test.ts and src/rbd.test.ts exists
- ✅ src/server.ts thin entry point exists
- ✅ package.json build and node:test scripts exists
- ✅ src/mountPointEntry.ts MountPointEntry.references exists

## Recommendation
Specify the safe behavior when Create’s final unMap fails, including its response and test. Otherwise the prerequisite structure is present, the dependency order is explicit, and the two-production-file scope is feasible.
