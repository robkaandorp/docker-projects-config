---
title: Review: ci-tests-and-dockerhub-publish
type: scratch
status: draft
author: reviewer
tags: []
links: []
created: 2026-09-28
updated: 2026-09-28
---

# Review: ci-tests-and-dockerhub-publish

## Verdict: NeedsChanges

## Issues

### [MAJOR] The prerequisite has not landed in the inspected repository: package.json has no test script and src/*.test.ts does not exist. Make the dependency on restructure-for-testability-and-add-tests enforceable in dispatch, and do not start this goal until it lands.

### [MAJOR] The already-released guard needs authenticated GitHub CLI access before Docker Hub publishing. Specify GH_TOKEN for the guard and require it to distinguish a missing release from authentication or API failures; otherwise a failed lookup could be mistaken for an unreleased version.

### [MINOR] “Develop and PR builds only build and test” conflicts literally with the requirement to export the image and create a tgz on every trigger. Clarify that these builds may package artifacts but must not publish them.

## Verified
- ✅ docker-volume-rbd/.github/workflows/docker-image.yml exists
- ✅ docker-volume-rbd/Dockerfile builder stage and pnpm build/prune steps exists
- ✅ docker-volume-rbd/build.sh exists
- ✅ docker-volume-rbd/README.md exists
- ✅ docker-volume-rbd/package.json exists
- ❌ package.json test script does not exist
- ❌ docker-volume-rbd/src/*.test.ts does not exist
- ❌ docker-volume-rbd/VERSION does not exist
- ✅ docker-volume-rbd/config.json, entrypoint.sh, tsconfig.json, pnpm-lock.yaml exists
- ✅ docker-volume-rbd/.github/copilot-instructions.md exists

## Recommendation
Enforce the stated prerequisite before dispatch. Specify fail-closed, authenticated release checking and clarify that non-master builds package but never publish. The remaining scope is feasible as one goal.
