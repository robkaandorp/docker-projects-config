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

## Verdict: Approved

## Issues

No issues found.

## Verified
- ✅ docker-volume-rbd/.github/workflows/docker-image.yml exists
- ✅ publish job, version guard, Docker Hub login, managed-plugin step, and final GitHub release step exists
- ✅ docker-volume-rbd/README.md Releases / CI paragraph exists
- ✅ docker-volume-rbd/.github/copilot-instructions.md Versioning and CI paragraph exists
- ✅ docker-volume-rbd/VERSION contains v20.2-r1 exists
- ✅ docker-volume-rbd/package.json defines pnpm test exists

## Recommendation
Proceed. The change is confined to one workflow step and two documentation sentences, with clear ordering and fail-closed verification criteria.
