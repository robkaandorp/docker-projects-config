---
title: Review: fix-publish-base-tag-retag
type: scratch
status: draft
author: reviewer
tags: []
links: []
created: 2026-09-29
updated: 2026-09-29
---

# Review: fix-publish-base-tag-retag

## Verdict: NeedsChanges

## Issues

### [CRITICAL] The proposed `docker buildx imagetools create` command conflicts with the required digest-equality check. For a single non-index source, buildx normally creates an index by default; `--prefer-index=false` is the option intended to preserve a single manifest, but the goal expressly forbids it. As written, the base tag may have a different manifest digest even if the command succeeds.

## Verified
- ✅ docker-volume-rbd/.github/workflows/docker-image.yml exists
- ✅ publish job, version guard, Docker Hub login, managed-plugin step, and last GitHub release step with --target "${{ github.sha }}" exists
- ✅ docker-volume-rbd/README.md Releases / CI paragraph exists
- ✅ docker-volume-rbd/.github/copilot-instructions.md Versioning and CI paragraph exists
- ✅ docker-volume-rbd/VERSION containing v20.2-r1 exists
- ✅ docker-volume-rbd/package.json pnpm test script exists

## Recommendation
Confirm buildx behavior for a Docker plugin manifest, then specify a registry-side copy command and flags that preserve the manifest bytes and digest. Remove the prohibition on `--prefer-index=false` if that flag is needed. The remaining three-file scope and verification requirements are appropriate.
