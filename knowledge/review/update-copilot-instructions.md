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

### [MAJOR] The goal is not ready for dispatch against the current checkout. The prerequisite changes have not landed: src/app.ts, src/config.ts, and src/*.test.ts are absent; package.json has no test script; and the workflow neither tests nor runs docker plugin push. Keep this goal blocked until its stated dependencies land, then verify the documentation against the resulting files.

## Verified
- ✅ .github/copilot-instructions.md exists
- ✅ src/server.ts exists
- ❌ src/config.ts / parseConfig does not exist
- ❌ src/app.ts / createApp does not exist
- ✅ src/rbd.ts / Rbd exists
- ✅ src/mountPointEntry.ts / MountPointEntry exists
- ❌ src/*.test.ts does not exist
- ✅ tsconfig.json exists
- ✅ Dockerfile exists
- ✅ package.json exists
- ✅ config.json exists
- ✅ .github/workflows/docker-image.yml exists

## Recommendation
Retain the documentation-only scope and source-verification rule. Dispatch only after the named prerequisites land; the proposed work should then be feasible as one full-file replacement.
