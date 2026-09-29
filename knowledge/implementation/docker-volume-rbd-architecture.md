---
title: docker-volume-rbd: architecture (as of v20.2-r1)
type: implementation
status: draft
author: composer
tags: [docker-volume-rbd, architecture, ci, release-v20.2-r1]
links:
  - target: memory-docker-volume-rbd-decisions
    type: related
    description: Owner decisions and conventions behind this architecture
created: 2026-09-29
updated: 2026-09-29
---

# docker-volume-rbd: architecture (as of v20.2-r1)

State of `robkaandorp/docker-volume-rbd` after release **v20.2-r1** (2026-09-29). It was released from master commit `2a21b52`, then tested and rolled out to the owner's swarm. Decisions and conventions live in `memory-docker-volume-rbd-decisions`; this document describes what is built.

## What it is
A Docker *managed plugin* implementing the Docker Volume Plugin API over `/run/docker/plugins/rbd.sock`. It maps Ceph RBD images, formats them (xfs by default) and mounts them under `/mnt/volumes/<pool>/<name>`, a `propagatedmount` in `config.json`. Published as `robkaandorp/rbd:v20.2` (moving tag) and `robkaandorp/rbd:v20.2-r1`.

## Source layout (`src/`)
| File | Role |
|---|---|
| `server.ts` | Thin entry point: `parseConfig(process.env)` → `new Rbd(config)` → `createApp(rbd, pool)` → `listen` on the socket. |
| `config.ts` | `parseConfig`: RBD_CONF_POOL (default `rbd`), RBD_CONF_CLUSTER / RBD_CONF_KEYRING_USER (`undefined` when unset or empty), RBD_CONF_MAP_OPTIONS (split on `;`, default `["--exclusive"]`, empty → `[]`). |
| `app.ts` | `createApp(rbd: RbdInterface, pool)`: all Express route handlers, per-instance `mountPointTable`, `createVolumeLock()` / `withVolumeLock` (per-volume, in-process promise chain), `cleanupBestEffort` rollback helper. |
| `rbd.ts` | `Rbd` wraps the `rbd`, `mkfs`, `mount` and `umount` CLIs through an injectable `CommandRunner` and an injectable `FileSystem` (existsSync, mkdirSync, rmdirSync, readFileSync). `commonArgs()` adds `--cluster` / `--id` only when set. `getMountedDevice()` parses `/proc/mounts` (exact target match). Remove = `rbd trash move`. Timeouts are 30 s, mkfs 120 s. |
| `mountPointEntry.ts` | Reference counting of container IDs per mountpoint. |
| `*.test.ts` | `node:test` suites: fake `CommandRunner`/`FileSystem` for `Rbd`; stub `RbdInterface` + `app.listen(0)` + `fetch` for the app. |

## Behaviour highlights
- **Err convention:** all `/VolumeDriver.*` endpoints return `Err` ("" on success). Errors are never thrown to Express.
- **Rollback (Mount/Create):** best-effort, and only undoes what the request itself did (ownership tracked per request, e.g. `ownsMapping`). Cleanup failures are logged and never replace the original error. A formatted image whose final unmap failed during Create is kept.
- **Per-volume lock:** Create/Remove/Mount/Unmount on the same volume run one at a time, within this process only. Cross-node protection is `rbd map --exclusive`, which is currently disabled in production.
- **Untracked volumes (after plugin restart):** no startup recovery.
  - Untracked Unmount: unmounts the image's own device if it is mounted, then unmaps. Success only if the cleanup succeeds.
  - Untracked Mount: adopts an existing mount of the same device.
  - A conflicting device at the mountpoint returns `Err` and changes nothing.
  - Tracked volumes use normal reference counting.

## Build & toolchain
- TypeScript: `strict`, `module nodenext`, `target esnext`, `lib es2023`, `types node`.
- Node major pinned in `.node-version` (24), with a matching `@types/node`. pnpm is pinned via `packageManager` (Corepack). Workers install Node with the `.github/skills/setup-node/` skill.
- `pnpm test` = `tsc && node --test dist/*.test.js`.
- Dockerfile: ubuntu:24.04 base with ceph-common from the Ceph Tentacle repo, Node (nodesource `setup_<.node-version>.x`), xfsprogs and kmod. The builder stage runs build → test → prune. `.dockerignore` excludes `.git/`.
- `build.sh`: manual local build/install, derives the base tag from `VERSION`.

## CI (`.github/workflows/docker-image.yml`)
- Workflow permissions default to `contents: read`.
- **`build`** job: runs on push/PR to `master` or `develop`. Checkout without persisted credentials, validate `VERSION`, docker build (runs the tests), export rootfs, tgz, upload artifact (1 day).
- **`publish`** job: master push only; `needs: build`; the only `contents: write`; concurrency group `publish-master` (no cancel); no checkout. Steps:
  1. Download the artifact.
  2. Fail-closed guard: the GitHub release and tag must both return 404.
  3. Docker Hub login.
  4. `docker plugin create` + `push` of `<full>` (once).
  5. `docker buildx imagetools create --prefer-index=false` retag to `<base>`, plus a digest-equality check.
  6. `gh release create <full> --target <sha>` with `GH_TOKEN` + `GH_REPO`, last.

## History
- Goals (all in release v20.2-r1):
  - fix-list-bug-and-harden-tsconfig
  - pin-node-24-lts-and-setup-node-skill
  - restructure-for-testability-and-add-tests
  - rollback-and-per-volume-locking
  - handle-untracked-volumes-after-restart
  - ci-tests-and-dockerhub-publish
  - update-copilot-instructions
  - fix-publish-base-tag-retag
  - fix-release-step-gh-repo
- The release needed two publish fixes found on real master runs: the double `plugin create`, and `gh` without a checkout.