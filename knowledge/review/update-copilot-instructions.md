---
title: Review: update-copilot-instructions
type: scratch
status: draft
author: reviewer
tags: []
links: []
created: 2026-09-28
updated: 2026-09-28
---

# Review: update-copilot-instructions

## Verdict: NeedsChanges

## Issues

### [MAJOR] The untracked-volume instructions say Unmount performs a best-effort unmount and unmap and returns success. In src/app.ts, a failed unmount prevents unmap, and any cleanup error returns a nonempty Err. Specify that it returns success only when the attempted cleanup succeeds; distinguish this from best-effort rollback.

### [MINOR] The workflow does not run on every push or pull request. .github/workflows/docker-image.yml limits both triggers to master and develop. Qualify the CI description accordingly.

## Verified
- ✅ .github/copilot-instructions.md and all listed files to inspect exists
- ✅ src/*.test.ts; package.json build and test scripts exists
- ✅ parseConfig, createApp, RbdInterface, createVolumeLock, withVolumeLock, cleanupBestEffort exists
- ✅ Rbd, CommandRunner, FileSystem, getMountedDevice, MountPointEntry exists
- ✅ .github/skills/setup-node/ and .github/workflows/docker-image.yml exists

## Recommendation
Correct the untracked-Unmount and CI-trigger descriptions before dispatch. The remaining documentation-only scope is feasible as one full-file replacement.
