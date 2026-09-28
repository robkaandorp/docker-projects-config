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

### [MAJOR] The goal depends on fix-list-bug-and-harden-tsconfig, but that work is not present in the repository and no explicit dependency is specified. Currently List uses `Name: name`, cluster/user default to `ceph`/`admin`, Rbd does not pass `--cluster` or `--id`, and tsconfig is not strict. The requested tests and “no behaviour changes” cannot both be satisfied against this checkout. Make the ordering explicit and dispatch only after that goal lands.

### [MINOR] The example test script quotes `dist/**/*.test.js`, while all requested tests are directly under `src/`. Specify a test command known to discover the compiled files in the supported Node version, such as `node --test dist/*.test.js`.

## Verified
- ✅ docker-volume-rbd/src/rbd.ts; Rbd.create, map, unMap, list, remove, mount, unmount, isMapped, getInfo, makeFilesystem exists
- ✅ docker-volume-rbd/src/server.ts; routes and mountPointTable exists
- ✅ docker-volume-rbd/src/mountPointEntry.ts; MountPointEntry.references and hasReference exists
- ❌ docker-volume-rbd/src/app.ts and src/config.ts does not exist
- ❌ docker-volume-rbd/src/*.test.ts does not exist
- ✅ docker-volume-rbd/package.json, pnpm-lock.yaml, tsconfig.json, entrypoint.sh exists

## Recommendation
Make the previous fix an explicit prerequisite, then confirm its config and CLI semantics before requiring behaviour-preserving tests. Clarify the test discovery command. The remaining refactor and small suite are appropriately scoped for one goal.
