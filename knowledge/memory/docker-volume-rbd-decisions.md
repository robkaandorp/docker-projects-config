---
title: docker-volume-rbd: owner decisions and conventions
type: memory
status: active
author: composer
tags: [docker-volume-rbd, decisions, branching, ci, versioning, node]
links:
  - target: implementation-docker-volume-rbd-architecture
    type: related
    description: Architecture of what was built under these decisions
created: 2026-09-28
updated: 2026-09-29
---

# docker-volume-rbd: project decisions

Things the owner (Rob Kaandorp) decided, to keep in mind when planning goals for `docker-volume-rbd`. For how the code is actually built, see `implementation-docker-volume-rbd-architecture`.

## Usage context
- The main user is the owner's own office Docker Swarm, so there are few users. Keep changes simple and lightweight. Don't over-engineer.

## Ceph version
- The cluster and the swarm hosts run **Ceph Tentacle 20.2**. The Dockerfile installs ceph-common from the official download.ceph.com `debian-tentacle` repo for noble.
- **Exclusive locking is turned off in production** (`RBD_CONF_MAP_OPTIONS` set to empty) because the exclusive-lock feature isn't enabled on the rbd images yet. The code default is still `--exclusive`. Check this setting after every plugin upgrade.

## Branching and publishing
- Development happens on **`develop`** (the GitHub default branch). Goals target `develop`.
- Releases are develop → master merges. Only a **push to `master`** publishes, to Docker Hub and as a GitHub release; develop and PR builds only build and test.
- It is a Docker *managed plugin*: it is published with `docker plugin create` / `push`, never `docker push`.
- Docker Hub secrets `DOCKERHUB_USERNAME` / `DOCKERHUB_TOKEN` are configured (2026-09-28).

## Versioning
- A `VERSION` file at the repo root holds `v<ceph major>.<minor>-r<revision>`, e.g. `v20.2-r1`.
- A release publishes `robkaandorp/rbd:<base>` (moving tag, e.g. `v20.2`, the one users install), `robkaandorp/rbd:<full>` (e.g. `v20.2-r1`), and a GitHub release + tag `<full>` on the built master commit.
- **Bump the revision on develop before each release merge.** CI fails closed if that GitHub release or tag already exists.
- A version counts as unreleased until its GitHub release exists. A rerun after a partial publish may overwrite the Docker Hub tags of that version; this is accepted (issue publish-guard-does-not-check-docker-hub-tags…, acknowledged).
- CopilotHive release IDs use the same scheme. CopilotHive's own tag-on-merge option stays **disabled**; the workflow is the only tag owner.
- This replaced the old `vars.VERSION_TAG` + `github.run_number` scheme (last old release: v20.2.26).

## Config options
- `RBD_CONF_CLUSTER` / `RBD_CONF_KEYRING_USER`: `--cluster` / `--id` are passed only when the variable is set and non-empty, so command lines are unchanged when unset. The owner doesn't use them today.

## Restart handling
- There is deliberately no startup recovery of the mount table. Untracked volumes are handled lazily when Docker asks about them.
- Accepted trade-off: after a restart, the first Unmount cleans the volume up even if another pre-restart container still uses it.
- The owner verified this on the swarm in 2026-09.

## Node.js version
- The Node **major** is pinned in `.node-version` (currently `24`). The Dockerfile derives nodesource `setup_<major>.x` from it, and the `@types/node` major matches it.
- Moving to the next LTS is a deliberate bump: `.node-version` + `@types/node` major + regenerating the lockfile with pnpm. Node 26 becomes LTS on 2026-10-28.
- Node 25+ no longer bundles Corepack, so a bump to 26 needs `npm i -g corepack` (or equivalent) in the Dockerfile and `.github/skills/setup-node/install-node.sh`.
- Worker images have no Node; workers use the `setup-node` repo skill.

## Testing
- Node's built-in test runner (`node:test`) only, no extra framework. Tests must not need Ceph, root or real binaries.

## CI publish job gotchas (learned during the v20.2-r1 release)
- The master `publish` job deliberately has no `actions/checkout` (it holds a write token and runs no repo code). So every `gh` subcommand other than `gh api repos/...` needs `GH_REPO: ${{ github.repository }}`; otherwise it fails with `fatal: not a git repository`.
- One Docker daemon cannot `docker plugin create` twice from the same rootfs ("content sha256:…: already exists"). Push once as `<full>`, then retag on the registry with `docker buildx imagetools create --prefer-index=false`. The flag is required: without it buildx wraps the manifest in a list.
- Workers can't run GitHub Actions or push to Docker Hub, so publish-path bugs only surface on a real master run. Review publish-step changes against real tool behaviour, not stubs.