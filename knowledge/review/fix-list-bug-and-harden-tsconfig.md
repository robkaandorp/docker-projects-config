---
title: Review: fix-list-bug-and-harden-tsconfig
type: scratch
status: draft
author: reviewer
tags: []
links: []
created: 2026-09-28
updated: 2026-09-28
---

# Review: fix-list-bug-and-harden-tsconfig

## Verdict: NeedsChanges

## Issues

### [MAJOR] docker-volume-rbd/pnpm-lock.yaml currently contains two separate YAML documents, each with an importers section. The first lists pnpm itself; the second lists the project's dependencies. Simply adding @types/node to the existing lockfile may leave the Docker build's `pnpm ci` step broken. Require a regenerated, valid lockfile and verification that `pnpm ci` succeeds, in addition to `pnpm run build`.

## Verified
- ✅ docker-volume-rbd/src/server.ts and /VolumeDriver.List handler using Name: name exists
- ✅ docker-volume-rbd/src/rbd.ts, Rbd.options, isMapped, map, unMap, list, create, remove, makeFilesystem, mount, unmount exists
- ✅ docker-volume-rbd/tsconfig.json exists
- ✅ docker-volume-rbd/package.json exists
- ✅ docker-volume-rbd/pnpm-lock.yaml exists
- ✅ docker-volume-rbd/Dockerfile and stale Ceph comment exists
- ✅ docker-volume-rbd/README.md and unimplemented-option notes exists
- ✅ docker-volume-rbd/src/mountPointEntry.ts, config.json, entrypoint.sh, build.sh, .github/workflows/docker-image.yml, .github/copilot-instructions.md exists

## Recommendation
Keep the proposed scope; it is feasible without a split and addresses the identified bugs directly. Amend the lockfile instructions and acceptance criteria to require a single valid regenerated lockfile and a successful `pnpm ci` before dispatch. No depends_on goals are specified or appear necessary.
