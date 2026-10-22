---
title: Review: drone-portainer-semver-releases
type: scratch
status: draft
author: reviewer
tags: []
links: []
created: 2026-10-22
updated: 2026-10-22
---

# Review: drone-portainer-semver-releases

## Verdict: Approved

## Issues

No issues found.

## Verified
- ✅ drone-portainer/.github/workflows/docker-image.yml exists
- ✅ Current workflow: master-only push/PR triggers, unconditional Docker publishing, VERSION_TAG/run_number tags, and existing Docker Hub secret references exists
- ✅ drone-portainer/README.md: Version history table and untagged usage image exists
- ✅ drone-portainer/.github/copilot-instructions.md: CI bullet at line 16 exists
- ✅ drone-portainer/package.json: version is 1.0.0 and scripts.build is tsc; no test script exists
- ✅ drone-portainer/pnpm-lock.yaml exists
- ✅ drone-portainer/tsconfig.json exists
- ✅ drone-portainer/Dockerfile exists
- ✅ drone-portainer/src/index.ts: opt-in self-signed certificate support exists
- ❌ drone-portainer/.github/skills does not exist

## Recommendation
Proceed as written within the drone-portainer repository. The three-file scope is feasible in 1–3 iterations, and the acceptance criteria and stubbed failure-path checks are sufficiently explicit. No split or additional dependencies are needed. Remote settings, secret availability, Docker Hub tags, and GitHub release inventory cannot be independently verified through the available local file tools.
