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

### [CRITICAL] The goal describes a restructuring that is not present in this checkout. src/config.ts and src/app.ts do not exist; src/server.ts still owns configuration, routes, and the mount table. There is no startup recovery, per-volume lock, or rollback implementation to document.

### [MAJOR] package.json has no test script and src contains no *.test.ts files. Rbd has no injectable command runner or filesystem, and MountPointEntry has no recovered or implicit reference. The requested testing and recovery guidance cannot truthfully describe the current code.

### [MAJOR] The requested config, API, and TypeScript claims conflict with the source: cluster and user are stored but not passed to rbd; /Plugin.Activate and /VolumeDriver.Capabilities do not return Err; tsconfig.json does not specify strict, lib, or types.

### [MAJOR] The Dockerfile builder runs build but not test. The workflow targets master for pushes and pull requests, creates a GitHub release, and does not run docker plugin push. The requested develop-branch CI and Docker Hub publishing description is not supported by these files.

### [MINOR] The Dockerfile configures the official Ceph Tentacle repository and installs ceph-common, but its preceding comment describes Ubuntu's Squid package. Documentation should distinguish the configured repository from that stale comment.

## Verified
- ✅ .github/copilot-instructions.md exists
- ✅ src/server.ts exists
- ❌ src/config.ts / parseConfig does not exist
- ❌ src/app.ts / createApp does not exist
- ✅ src/rbd.ts / Rbd exists
- ❌ Rbd injectable command runner and filesystem does not exist
- ✅ src/mountPointEntry.ts / MountPointEntry exists
- ❌ MountPointEntry implicit-reference policy does not exist
- ❌ src/*.test.ts and package.json test script does not exist
- ✅ package.json, pnpm-lock.yaml, tsconfig.json, Dockerfile, config.json, entrypoint.sh, build.sh, README.md, .github/workflows/docker-image.yml exists

## Recommendation
Keep this as a one-file documentation goal, but dispatch it after the referenced restructuring, tests, robustness, and CI changes land. Alternatively, revise the required content to document this checkout as it stands. No dependency ordering is specified; if those changes are separate goals, make them explicit prerequisites.
