---
title: Review: restructure-for-testability-and-add-tests
type: scratch
status: draft
author: reviewer
tags: []
links: []
created: 2026-09-28
updated: 2026-09-28
---

# Review: restructure-for-testability-and-add-tests

## Verdict: NeedsChanges

## Issues

### [CRITICAL] The prerequisite has not landed in the inspected docker-volume-rbd checkout. tsconfig.json does not enable strict mode or remove the DOM lib; server.ts defaults cluster/user to "ceph"/"admin"; Rbd does not pass --cluster or --id; and VolumeDriver.List uses Name: name rather than info.image. Dispatch only after fix-list-bug-and-harden-tsconfig is applied, so the requested no-behaviour-change baseline exists.

## Verified
- ✅ docker-volume-rbd/src/rbd.ts and Rbd methods create, map, unMap, list, remove, mount, unmount exists
- ✅ docker-volume-rbd/src/server.ts and VolumeDriver routes exists
- ✅ docker-volume-rbd/src/mountPointEntry.ts and MountPointEntry.hasReference exists
- ✅ docker-volume-rbd/package.json, pnpm-lock.yaml, tsconfig.json, and entrypoint.sh exists
- ❌ docker-volume-rbd/src/app.ts does not exist
- ❌ docker-volume-rbd/src/config.ts does not exist
- ❌ docker-volume-rbd/src/*.test.ts does not exist
- ❌ Prerequisite strict tsconfig, unset cluster/user semantics, CLI flags, and List image-name fix does not exist

## Recommendation
Keep the goal and its dependency order, but wait for the prerequisite to land and verify its semantics before dispatch. The proposed refactor and test suite are otherwise appropriately scoped and testable.
