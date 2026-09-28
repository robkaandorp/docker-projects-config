---
title: Review: recover-mount-table-on-startup
type: scratch
status: draft
author: reviewer
tags: []
links: []
created: 2026-09-28
updated: 2026-09-28
---

# Review: recover-mount-table-on-startup

## Verdict: NeedsChanges

## Issues

### [MAJOR] The goal assumes prior refactors that are not present in docker-volume-rbd: src/app.ts and createApp do not exist; route handlers and mountPointTable are in src/server.ts. Rbd has no injectable command runner, and per-volume serialization is not implemented. Specify the prerequisite goals and their ordering, or explicitly include this work in the scope.

### [MAJOR] The stated test acceptance criteria cannot currently be run: there are no src/*.test.ts files and package.json has no test script. Include test setup in the scope or make it an explicit prerequisite.

### [MINOR] Clarify what should happen if recovery finds a mount but a later Unmount ID is unknown after the single implicit reference has already been released. The proposed one-reference policy cannot account for multiple pre-restart containers; the requested code comment should make that operational risk clear.

## Verified
- ❌ docker-volume-rbd/src/app.ts and createApp does not exist
- ✅ docker-volume-rbd/src/rbd.ts and Rbd.isMapped exists
- ❌ Rbd.listMapped and injectable command runner does not exist
- ✅ docker-volume-rbd/src/server.ts, mountPointTable, and getMountPoint exists
- ✅ docker-volume-rbd/src/mountPointEntry.ts, MountPointEntry.references, and hasReference exists
- ❌ docker-volume-rbd/src/*.test.ts does not exist
- ❌ docker-volume-rbd/package.json test script does not exist

## Recommendation
Dispatch the assumed app extraction, runner injection, serialization, and test setup as ordered prerequisites. Then dispatch recovery against the resulting codebase; alternatively, revise and split this goal to include those changes explicitly.
