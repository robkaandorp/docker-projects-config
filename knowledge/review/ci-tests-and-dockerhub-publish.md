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

### [MAJOR] The release and tag checks are not atomic. Concurrent master runs using the same VERSION can both pass the guard, then publish conflicting Docker tags or create a release for only one of the built commits. Require serialization of master publish runs, without canceling a publish already in progress.

### [MAJOR] Granting contents: write to the job that builds pull-request code gives that job more permission than it needs. Specify how write permission will be confined to master publishing, or explicitly document the repository controls relied on to make this acceptable.

## Verified
- ✅ docker-volume-rbd/.github/workflows/docker-image.yml exists
- ✅ docker-volume-rbd/Dockerfile exists
- ✅ docker-volume-rbd/.dockerignore exists
- ✅ docker-volume-rbd/build.sh exists
- ✅ docker-volume-rbd/README.md exists
- ✅ docker-volume-rbd/package.json test script as specified exists
- ✅ docker-volume-rbd/src/*.test.ts exists
- ✅ docker-volume-rbd/.github/skills/setup-node exists
- ❌ docker-volume-rbd/VERSION (to be added) does not exist

## Recommendation
The requested files and test prerequisite are present, and the work is cohesive. Add a non-canceling concurrency requirement for master publishing and resolve the pull-request write-permission risk before dispatch.
