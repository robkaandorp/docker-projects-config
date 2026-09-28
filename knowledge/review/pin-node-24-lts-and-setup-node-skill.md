---
title: Review: pin-node-24-lts-and-setup-node-skill
type: scratch
status: draft
author: reviewer
tags: []
links: []
created: 2026-09-28
updated: 2026-09-28
---

# Review: pin-node-24-lts-and-setup-node-skill

## Verdict: Approved

## Issues

### [MINOR] There is no test script or configured test suite in package.json; the requirement that existing tests pass is effectively satisfied by the specified clean install and build.

### [MINOR] The installer must put its newly installed bin directory on PATH before invoking corepack; otherwise Corepack's Node-based executable may fail in the required no-Node environment. The matching-Node early-exit branch also does not guarantee that pnpm is available, so the skill should not imply that it does.

## Verified
- ✅ docker-volume-rbd/Dockerfile and base-stage setup_lts.x command exists
- ✅ docker-volume-rbd/package.json: @types/node ^22.20.4 and packageManager pnpm@12.4.2 exists
- ✅ docker-volume-rbd/pnpm-lock.yaml: two YAML documents exists
- ✅ docker-volume-rbd/tsconfig.json: strict TypeScript configuration exists
- ✅ docker-volume-rbd/.github/workflows/docker-image.yml exists
- ✅ docker-volume-rbd/.github/copilot-instructions.md exists
- ❌ docker-volume-rbd/.node-version does not exist
- ❌ docker-volume-rbd/.github/skills/setup-node/SKILL.md does not exist
- ❌ docker-volume-rbd/.github/skills/setup-node/install-node.sh does not exist

## Recommendation
Proceed as one goal. Ensure the script bootstraps Corepack with the downloaded Node on PATH, and treat the build—not a nonexistent test command—as the available project verification.
