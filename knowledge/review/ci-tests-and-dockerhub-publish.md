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

### [MAJOR] The workflow will have `contents: write`, but `actions/checkout` persists its token by default. The Dockerfile's `COPY . .` can copy checkout credentials from `.git` into the image, which is then exported and published as a plugin. Require `persist-credentials: false` on checkout, or another explicit protection that keeps the token out of the build context.

### [MINOR] “Tgz as a local job artifact” is ambiguous. The specified steps create a tgz on the runner but do not upload a downloadable GitHub Actions artifact. Clarify which is intended.

### [MINOR] The release guard checks for an existing release, not an existing git tag. If the full-version tag already exists at another commit, `gh release create --target` will not retarget it. Specify whether the job should verify or reject an existing tag to guarantee that the release tag points to the built master commit.

## Verified
- ✅ docker-volume-rbd/.github/workflows/docker-image.yml exists
- ✅ docker-volume-rbd/Dockerfile builder stage exists
- ✅ docker-volume-rbd/build.sh exists
- ✅ docker-volume-rbd/README.md exists
- ✅ docker-volume-rbd/package.json test script exists
- ✅ docker-volume-rbd/src/*.test.ts exists
- ❌ docker-volume-rbd/VERSION (proposed new file) does not exist
- ✅ docker-volume-rbd/config.json exists
- ✅ docker-volume-rbd/.github/copilot-instructions.md exists

## Recommendation
The prerequisite test script and test files are present, and the work is otherwise appropriately sized. Add a checkout credential-safety requirement, clarify whether the tgz must be uploaded as an Actions artifact, and define handling for a pre-existing full-version git tag.
