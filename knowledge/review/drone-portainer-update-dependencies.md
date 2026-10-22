---
title: Review: drone-portainer-update-dependencies
type: scratch
status: draft
author: reviewer
tags: []
links: []
created: 2026-10-22
updated: 2026-10-22
---

# Review: drone-portainer-update-dependencies

## Verdict: NeedsChanges

## Issues

### [MAJOR] Dockerfile line 7 runs `pnpm ci`, whereas the acceptance criteria validate `pnpm install --frozen-lockfile`. There is no `ci` script in package.json, and `pnpm ci` is not the established pnpm equivalent of npm ci. Requiring every Dockerfile line except the base-image tags to remain unchanged can prevent the worker from satisfying the Docker build criterion. Permit replacing this line with `RUN pnpm install --frozen-lockfile`, or establish that pnpm 12.15.0 supports the existing command.

### [MINOR] The repository confirms the existing dependency versions, but it cannot substantiate the requested releases, advisory affected ranges, or the assertion that the typescript package at 7.0.2 supplies the native compiler under the unchanged tsc binary. Require registry/advisory verification before editing, with an explicit instruction to report a blocker rather than substitute versions or fabricate integrity hashes if a specified release is unavailable.

## Verified
- ✅ drone-portainer/package.json exists
- ✅ package.json fields: version=1.0.0, main=dist/server.js, dependencies.axios=^1.17.0, devDependencies.typescript=^6.0.3, devDependencies.@types/node=^24.13.1, packageManager=pnpm@11.5.2 with integrity hash exists
- ✅ package.json scripts.build=tsc exists
- ✅ drone-portainer/pnpm-lock.yaml exists
- ✅ drone-portainer/Dockerfile: builder and dist stages use node:lts-alpine; CMD runs /app/dist/index.js exists
- ✅ drone-portainer/tsconfig.json: module=nodenext, rootDir=./src, outDir=./dist, types=[node] exists
- ✅ drone-portainer/src/index.ts exists
- ✅ drone-portainer/.github/workflows/docker-image.yml exists
- ✅ drone-portainer/README.md exists
- ✅ drone-portainer/.github/copilot-instructions.md exists
- ✅ drone-portainer/.dockerignore exists
- ✅ drone-portainer/.gitignore exists

## Recommendation
Allow the Docker dependency-install command to be corrected if necessary, and add a preflight check for the specified releases and security advisory ranges. Otherwise, the scope is appropriate for 1–3 iterations, the main-entry correction addresses the root cause, and the compilation/CommonJS checks are suitable without running the live plugin. Keep release-workflow and metadata changes separate as proposed.
