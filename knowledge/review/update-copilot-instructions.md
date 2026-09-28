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

## Verdict: Approved

## Issues

No issues found.

## Verified
- ✅ .github/copilot-instructions.md; src/server.ts, config.ts, app.ts, rbd.ts, mountPointEntry.ts, and *.test.ts exists
- ✅ tsconfig.json, package.json, Dockerfile, .dockerignore, .node-version, VERSION, build.sh, config.json exists
- ✅ .github/skills/setup-node/ and .github/workflows/docker-image.yml exists
- ✅ parseConfig, Rbd, createApp, RbdInterface, createVolumeLock, withVolumeLock, cleanupBestEffort, CommandRunner, FileSystem, getMountedDevice, MountPointEntry exists
- ✅ Untracked-volume behavior, rollback, locking, endpoint response shapes, and command timeouts described in the goal exists

## Recommendation
Proceed with the documentation-only replacement. The referenced files and code behavior match the goal; keep the resulting instructions concise and grounded in the current files.
