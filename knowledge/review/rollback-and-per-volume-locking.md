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

### [MAJOR] The prerequisite has not landed in docker-volume-rbd: src/app.ts and src/*.test.ts do not exist, src/server.ts still contains the route handlers, Rbd has no injectable runner or fs functions, and package.json has no test script. Do not dispatch this goal until `restructure-for-testability-and-add-tests` is present; the permitted files cannot establish that prerequisite.

### [MAJOR] The ownership rule for Create is too broad. After create succeeds, a failed map does not establish that this request owns a mapping; another process could have mapped the new image. Calling unMap based only on isMapped could undo that other mapping. Clarify that the guarantee is limited to this process, or specify how externally created mappings are protected.

### [MINOR] The opening guarantee is stronger than the proposed behavior: best-effort cleanup can leave a mapping behind, and an in-process lock cannot serialise requests across plugin processes. State these limits explicitly.

## Verified
- ❌ docker-volume-rbd/src/app.ts / createApp does not exist
- ❌ docker-volume-rbd/src/*.test.ts does not exist
- ✅ docker-volume-rbd/src/server.ts and VolumeDriver.Create, Remove, Mount, Unmount handlers exists
- ✅ docker-volume-rbd/src/rbd.ts / Rbd, isMapped, map, unMap, create, makeFilesystem, remove, mount exists
- ❌ Rbd injectable command runner and fs functions does not exist
- ❌ docker-volume-rbd/package.json / test script does not exist
- ✅ docker-volume-rbd/src/mountPointEntry.ts / references and hasReference exists
- ❌ docker-volume-rbd/src/config.ts does not exist

## Recommendation
Wait for and verify the prerequisite before dispatch. Then clarify mapping ownership under external concurrency and limit the stated guarantee to in-process serialisation and best-effort rollback. The proposed two-production-file scope is otherwise appropriate.
