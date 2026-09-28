---
title: Review: handle-untracked-volumes-after-restart
type: scratch
status: draft
author: reviewer
tags: []
links: []
created: 2026-09-28
updated: 2026-09-28
---

# Review: handle-untracked-volumes-after-restart

## Verdict: Approved

## Issues

### [MINOR] Clarify that an untracked Unmount must not call unMap if unmount fails; unmapping a potentially mounted device would be unsafe.

### [MINOR] Existing app tests assert exact Rbd call sequences. Adding getMountedDevice will require updating those assertions, not only the tests expecting "Unknown volume".

## Verified
- ✅ src/app.ts: createApp, RbdInterface, mountPointTable, createVolumeLock/withVolumeLock, cleanupBestEffort, ownsMapping, Mount and Unmount handlers exists
- ✅ src/rbd.ts: Rbd, CommandRunner, injectable FileSystem, isMapped, unmount and unMap exists
- ❌ src/rbd.ts: getMountedDevice and FileSystem.readFileSync (planned additions) does not exist
- ✅ src/app.test.ts: StubRbd with recorded calls and failWith exists
- ✅ src/rbd.test.ts: FakeFileSystem exists
- ✅ src/config.test.ts, src/server.ts, src/mountPointEntry.ts and listed protected files exists
- ✅ .github/skills/setup-node and package.json build/test scripts exists

## Recommendation
Proceed as one goal. Specify fail-fast behavior after an unmount error and update existing exact-call assertions alongside the new tests.
