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

### [CRITICAL] The prerequisite has not landed in the current repository. src/app.ts and src/*.test.ts do not exist, handlers are still in src/server.ts, Rbd has no injectable command runner or fs functions, and package.json has no test script. The goal cannot be completed within its file restrictions or tested as specified until restructure-for-testability-and-add-tests lands.

## Verified
- ❌ src/app.ts / createApp does not exist
- ❌ src/*.test.ts does not exist
- ❌ package.json pnpm test script does not exist
- ✅ src/server.ts / Create, Remove, Mount and Unmount handlers exists
- ✅ src/rbd.ts / Rbd, isMapped, map, unMap, create, makeFilesystem, remove, mount exists
- ❌ Rbd injectable command runner and fs functions does not exist
- ✅ src/mountPointEntry.ts / MountPointEntry.references exists

## Recommendation
Keep the stated prerequisite as a hard dispatch dependency and re-review after it lands. At that point, verify the new app, injection points, tests and test script before assigning this goal.
