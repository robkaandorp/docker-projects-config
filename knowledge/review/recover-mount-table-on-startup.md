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

### [MAJOR] The stated prerequisites have not landed in the checked-out repository. src/app.ts and src/*.test.ts do not exist, server.ts still owns the routes and mountPointTable, Rbd has no injectable runner, and package.json has no test script. Do not dispatch until both prerequisite goals are verified as landed.

### [MAJOR] Recovery depends on old mounts appearing in the restarted plugin's /proc/mounts. config.json declares /mnt/volumes as a propagated mount, but the goal does not establish that those mounts remain visible in a new plugin mount namespace. Validate this restart behavior; otherwise recovery may silently find no entries.

### [MINOR] src/config.ts is listed as a file not to change, but it does not exist in this repository.

## Verified
- ✅ src/server.ts exists
- ✅ src/rbd.ts and Rbd.isMapped() exists
- ✅ src/mountPointEntry.ts and MountPointEntry.hasReference() exists
- ✅ mountPointTable and getMountPoint() in src/server.ts exists
- ✅ Unknown volume and Unknown caller id responses exists
- ❌ src/app.ts and createApp() does not exist
- ❌ Rbd.listMapped() does not exist
- ❌ src/*.test.ts does not exist
- ❌ package.json test script does not exist
- ❌ src/config.ts does not exist

## Recommendation
Keep the goal gated on both prerequisites. Verify the post-prerequisite structure and that mounts survive visibly across a real plugin restart before dispatching; remove the nonexistent file from the exclusion list.
