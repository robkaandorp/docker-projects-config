---
title: drone-portainer: owner decisions and conventions
type: memory
status: active
author: composer
tags: [drone-portainer, decisions, versioning, ci]
links:
  - target: memory-docker-volume-rbd-decisions
    type: related
    description: Same owner conventions (develop branch, CI-owned tags) and CI lessons
created: 2026-10-22
updated: 2026-10-23
---

# drone-portainer: project decisions

Things the owner (Rob Kaandorp) decided, to keep in mind when planning goals for `drone-portainer`. It's a Drone CI plugin that deploys a compose stack to Portainer; the image is `robkaandorp/drone-portainer`.

## Branching and publishing (decided 2026-10)
- Development happens on **`develop`**, which is now the GitHub default branch. Releases are develop → master merges.
- Only a **push to `master`** publishes, to Docker Hub and as a GitHub release. Develop and PR builds only build.
- The Docker Hub secrets `DOCKERHUB_USERNAME` / `DOCKERHUB_TOKEN` exist.

## Versioning (decided 2026-10)
- **Semantic versioning from `package.json` `version`**; the tag and release are `v<version>`. The first release is **v1.0.0**.
- The old scheme `vars.VERSION_TAG` (`v2.16`) + `.github.run_number` encoded the *Portainer* version of the time. The owner dropped it because the Portainer API has been stable for a long time. The old tags `v2.16` / `v2.16.20`…`v2.16.27` remain on Docker Hub but are no longer updated.
- A master release publishes `:v<version>` and `:latest`. Bump the version on develop before each release merge; CI fails closed if that version was already released.
- CopilotHive release ID: `drone-portainer-v1.0.0` (tag `v1.0.0`). As with docker-volume-rbd, the workflow should be the only tag owner, so keep CopilotHive tag-on-merge disabled for this repo.

## Lessons carried over from docker-volume-rbd
- A release/publish job without a checkout needs `GH_REPO` for `gh release create`, and `--target <sha>` because the default branch is develop.



## Runtime and tooling (decided 2026-10-22/23)
- **Node.js pinned to 24 LTS** (`node:24-alpine` in both Dockerfile stages; `@types/node` ^24). Do not use the floating `node:lts-alpine`: it switched to Node 26 on 2026-10-28. Moving to Node 26 is a deliberate, separate future goal (Dockerfile + `@types/node` together).
- The owner prefers the **latest stable majors** for dev tooling: TypeScript 7 (native compiler) and pnpm 12. The TypeScript 7 native binary is statically linked, so it works on the Alpine builder. If a future TS 7.x breaks on musl, switch only the builder stage to `node:24-slim` (see acknowledged issue `typescript-7-native-compiler-package-declares-no-libc-field-so-alpine-installs-the-glibc-named-platform-package`).
- `package.json` `main` = `dist/index.js` (the build emits `src/index.ts` → `dist/index.js`).

## Changelog and release metadata (decided 2026-10-23)
- `CHANGELOG.md` uses the Keep a Changelog 1.1.0 format with an `[Unreleased]` section. On each version bump in `package.json` (on develop, before the master merge), move the `[Unreleased]` entries under the new version and add its compare/tag link references.
- The README "Version history" table keeps a one-line summary per version and links to `CHANGELOG.md`.
- The owner bundles README/changelog updates into a separate **release-metadata goal** at the end of a release, rather than into each feature goal.
- The workflow's version check is strict semver: `^(0|[1-9][0-9]*)\.(0|[1-9][0-9]*)\.(0|[1-9][0-9]*)$` after trimming surrounding whitespace.

## v1.0.0 (prepared 2026-10-23)
- Goals: `drone-portainer-semver-releases`, `drone-portainer-update-dependencies`, `drone-portainer-release-metadata`. All three are merged to develop (head 977fc98).
- Contents: axios 1.20.0 security fix, semver/GitHub releases, Node 24 pin, TS 7 / pnpm 12, `main` fix. Plugin behaviour is unchanged from v2.16.27.
- Publishing happens when the owner merges develop → master.