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

### [MAJOR] The target repository has no test script or test files. Adding RUN pnpm test will fail until restructure-for-testability-and-add-tests is completed. Make that goal an explicit prerequisite and dispatch this goal only after its test script is present.

### [MAJOR] GitHub release creation needs contents: write permission for GITHUB_TOKEN. The workflow does not declare it, so publication may fail when repository defaults grant read-only access. Specify the required workflow or job permission.

### [MINOR] The release and plugin tags use github.run_number, which remains the same on a workflow rerun. A rerun after partial publication may fail because a release or plugin tag already exists. Clarify whether reruns must be supported or whether failure is acceptable.

## Verified
- ✅ docker-volume-rbd/.github/workflows/docker-image.yml exists
- ✅ docker-volume-rbd/Dockerfile builder stage and specified build/prune steps exists
- ✅ docker-volume-rbd/README.md exists
- ✅ docker-volume-rbd/build.sh exists
- ✅ docker-volume-rbd/config.json exists
- ❌ docker-volume-rbd/package.json test script does not exist

## Recommendation
Keep the three-file scope, but make the test-setup goal a hard dependency, require contents: write for release creation, and state the expected behavior on partial-publication reruns.
