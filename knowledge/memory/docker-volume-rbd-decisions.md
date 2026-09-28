---
title: docker-volume-rbd: owner decisions and conventions
type: memory
status: draft
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
