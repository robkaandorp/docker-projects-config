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

### [MAJOR] The prerequisites have not landed in the checked-out docker-volume-rbd code: src/app.ts and src/*.test.ts are absent, handlers and mountPointTable remain in src/server.ts, Rbd has no injectable runner, and package.json has no test script. Do not dispatch this goal until both named prerequisites are merged and their resulting structure is verified.

### [MINOR] The recovery tests should cover a device mounted at the wrong path, not just mapped-but-unmounted images. Recovery must match both the mapped device and the expected mountpoint to avoid treating an unrelated mount as this volume.

## Verified
- ❌ docker-volume-rbd/src/app.ts does not exist
- ✅ docker-volume-rbd/src/server.ts and mountPointTable exists
- ✅ docker-volume-rbd/src/server.ts getMountPoint exists
- ✅ docker-volume-rbd/src/rbd.ts Rbd.isMapped exists
- ❌ docker-volume-rbd/src/rbd.ts Rbd.listMapped does not exist
- ✅ docker-volume-rbd/src/mountPointEntry.ts MountPointEntry.references and hasReference exists
- ❌ docker-volume-rbd/src/*.test.ts does not exist
- ❌ docker-volume-rbd/package.json test script does not exist

## Recommendation
Keep the prerequisite ordering, but block dispatch until both prerequisite goals land. Then recheck the stated API and test setup, and add a wrong-mountpoint recovery test. The proposed implementation scope is otherwise reasonable.
