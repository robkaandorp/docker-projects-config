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

## Verdict: NeedsChanges

## Issues

### [CRITICAL] The required prerequisites have not landed in the inspected repository. src/app.ts, src/config.ts, src/*.test.ts, and the pnpm test script are absent; handlers and mountPointTable remain in src/server.ts. Do not dispatch this goal until both prerequisite goals have landed.

### [MAJOR] Unknown-volume Unmount would unmount any device found at the mountpoint, without checking that it is the device mapped for the requested image. A conflicting mount could therefore be unmounted. Specify conflict handling and add a test before allowing cleanup.

### [MINOR] The proposed lack of recovered reference counts means the first Unmount after a restart may attempt cleanup while another caller still uses the volume. This is a deliberate trade-off, but the operational risk should be stated explicitly.

## Verified
- ❌ src/app.ts / createApp does not exist
- ❌ src/config.ts does not exist
- ❌ src/*.test.ts does not exist
- ✅ src/server.ts / VolumeDriver.Mount and VolumeDriver.Unmount handlers exists
- ✅ mountPointTable in src/server.ts exists
- ✅ src/rbd.ts / Rbd, isMapped, unMap, mount, unmount exists
- ❌ Rbd.getMountedDevice does not exist
- ✅ src/mountPointEntry.ts / MountPointEntry.hasReference exists
- ❌ package.json / pnpm test script does not exist
- ✅ config.json, Dockerfile, entrypoint.sh, build.sh, README.md, pnpm-lock.yaml, .github/workflows/docker-image.yml, .github/copilot-instructions.md exists

## Recommendation
Wait for both prerequisites, then recheck the stated structure. Define safe behavior when an unknown volume's mountpoint contains a different device, and test that case.
