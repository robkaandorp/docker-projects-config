---
title: docker-volume-rbd: owner decisions and conventions
type: memory
status: active
author: composer
tags: [docker-volume-rbd, decisions, branching, ci]
links: []
created: 2026-09-28
updated: 2026-09-28
---

# docker-volume-rbd: project decisions

Things the owner (Rob Kaandorp) decided, to keep in mind when planning goals for `docker-volume-rbd`.

## Usage context
- The main user is the owner's own office Docker Swarm, so there are few users. Keep changes simple and lightweight. Don't over-engineer.

## Ceph version
- The cluster and the swarm hosts run **Ceph Tentacle 20.2**. The Dockerfile installs ceph-common from the official download.ceph.com `debian-tentacle` repo for noble. This works on the hosts. The old Dockerfile comment about needing Squid 19.2 for kernel 5.10 is **out of date**.
- **Exclusive locking is currently turned off** in production (`RBD_CONF_MAP_OPTIONS` set to empty), because the exclusive-lock feature isn't enabled on the rbd images yet. The code default is still `--exclusive`.

## Branching and publishing
- Development happens on a **`develop`** branch. Goals should target `develop`.
- Publishing to Docker Hub (`docker plugin push robkaandorp/rbd:<tag>`) and creating the GitHub release must only happen on a **push to `master`**, i.e. after a release merge from develop to master. Develop and PR builds only build and test.
- It is a Docker *managed plugin*, so it is published with `docker plugin push`, not `docker push`.

## Config options
- `RBD_CONF_CLUSTER` / `RBD_CONF_KEYRING_USER`: **implement, fully backwards compatible**. Only pass `--cluster` / `--id` to rbd when the variable is explicitly set and non-empty. When unset, the command lines must be unchanged. The owner doesn't use them today.

## Testing
- Use Node's built-in test runner (`node --test`), with no extra test framework. Restructure for testability: an injectable command runner in `Rbd`, and a `createApp()` factory split from the socket `listen` entry point.



## Versioning (decided 2026-09-28)
- A `VERSION` file at the repo root holds `v<ceph major>.<minor>-r<revision>`, e.g. `v20.2-r1`.
- Each master release publishes Docker Hub tags `robkaandorp/rbd:v20.2` (moving tag, the one users install) and `robkaandorp/rbd:v20.2-r1` (immutable), plus a GitHub release `v20.2-r1`.
- Bump the revision on develop before each release merge. CI fails on master if that version was already released.
- CopilotHive release IDs/tags follow the same scheme (first release: `v20.2-r1`). The owner turned off CopilotHive's automatic tagging on merge to master, because CI does the tagging.
- This replaces the old `vars.VERSION_TAG` + `github.run_number` scheme (the last old release was v20.2.26).

## Restart handling (decided 2026-09-28)
- No startup recovery of the mount table. Untracked volumes are handled when Docker asks about them: Unmount of an untracked volume unmounts and unmaps it if its own device is mounted; Mount adopts an existing mount of the same device. A conflicting device at the mountpoint returns an Err and nothing is changed.



## Tag ownership (decided 2026-09-28)
- CopilotHive's "tag on release / merge to master" option stays **disabled** for docker-volume-rbd. The GitHub Actions workflow is the only thing that creates release tags: `gh release create "$FULL_VERSION" --target "${{ github.sha }}"` on push to master.
- `--target` is required because GitHub's default branch is `develop`. Without it, `gh release create` would put a new tag on develop's HEAD.