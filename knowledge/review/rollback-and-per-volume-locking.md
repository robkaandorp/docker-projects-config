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

## Verdict: Approved

## Issues

### [MINOR] The claim that --exclusive is disabled in production cannot be verified from this repository; the code defaults to --exclusive, and a deployment can override it.

## Verified
- ✅ src/app.ts: createApp and Create, Remove, Mount, Unmount handlers exists
- ✅ src/rbd.ts: Rbd, CommandRunner, injectable FileSystem, and referenced Rbd methods exists
- ❌ src/rbd.ts: FileSystem.existsSync does not exist
- ✅ src/server.ts: thin createApp entry point exists
- ✅ src/app.test.ts: StubRbd with failWith exists
- ✅ src/rbd.test.ts and src/config.test.ts: node:test tests exists
- ✅ .github/skills/setup-node/SKILL.md exists

## Recommendation
Proceed as scoped. Add existsSync to the FileSystem interface and its test fake, and keep the specified rollback ownership and final-unMap safeguards.
