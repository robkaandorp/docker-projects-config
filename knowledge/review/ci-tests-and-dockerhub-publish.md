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

### [MAJOR] The specified GitHub Actions concurrency group serializes running publish jobs but does not guarantee that every master push publishes. GitHub Actions permits only one pending run per group; a newer pending run can replace an older one even with cancel-in-progress: false. This conflicts with the requirement that master pushes publish automatically.

## Verified
- ✅ docker-volume-rbd/.github/workflows/docker-image.yml exists
- ✅ docker-volume-rbd/Dockerfile exists
- ✅ docker-volume-rbd/.dockerignore exists
- ✅ docker-volume-rbd/build.sh exists
- ✅ docker-volume-rbd/README.md exists
- ❌ docker-volume-rbd/VERSION (new file) does not exist
- ✅ package.json test script matching the prerequisite exists
- ✅ src/*.test.ts exists
- ✅ .github/skills/setup-node exists

## Recommendation
Clarify whether skipping a queued master release is acceptable. If every master push must publish, specify a queueing approach that preserves every publish run; otherwise document the concurrency limitation. The remaining scope and file references are feasible.
