---
title: Progress: drone-portainer-update-dependencies
type: scratch
status: draft
author: composer
tags: []
links: []
created: 2026-10-22
updated: 2026-10-22
---

# Progress: drone-portainer-update-dependencies

## Iteration 1

### Brain Plan

Phases: Coding → Testing → Review → Merging

Reasoning: First clean iteration of a well-scoped dependency/bugfix goal: three coordinated edits in package.json (axios security bump, TypeScript 7, packageManager via Corepack, main entry fix), lockfile regeneration with pnpm 12.15.0, Dockerfile base-image pinning in both stages with a possible install-command swap, and conditional-only changes to tsconfig.json/src/index.ts. The work is small and tightly coupled (lockfile must match package.json), so one coding round suffices. No documentation phase — the goal explicitly assigns README/changelog to a separate release-metadata goal. No improve phase — this is a clean first iteration with no prior feedback. Testing validates the acceptance criteria (frozen-lockfile install, TS7 build producing dist/index.js, node --check, CommonJS require of axios, pnpm audit, lts-alpine grep), then review and merging.

**Coding**: Repository: drone-portainer (repo root is drone-portainer/, e.g. drone-portainer/package.json). Read package.json, Dockerfile, tsconfig.json and src/index.ts before editing.

Step 0 — preflight: verify the target releases exist before touching anything: `npm view axios@1.20.0 version`, `npm view typescript@7.0.2 version bin`, `npm view pnpm@12.15.0 version`, `npm view @types/node@24 version`. Optionally run `pnpm audit --prod` on the current lockfile to confirm the axios advisory range. If any specified release is unavailable or the npm registry is unreachable, STOP and report a blocker — do NOT substitute other major versions and never fabricate integrity hashes.

1) package.json (change ONLY these fields):
- `main`: `dist/server.js` -> `dist/index.js`
- `dependencies.axios`: `^1.17.0` -> `^1.20.0`
- `devDependencies.typescript`: `^6.0.3` -> `^7.0.2`
- `devDependencies.@types/node`: stay on major 24, use latest 24.x (>= `^24.19.2`). Do NOT move to 25/26.
- `packageManager`: update to pnpm 12.15.0. Generate the field with Corepack (e.g. `corepack use pnpm@12.15.0`) so the sha512 integrity hash is correct — never hand-write the hash.
- `version` stays `1.0.0`; do not change any other field.

2) pnpm-lock.yaml: regenerate with pnpm 12.15.0 so it matches the new package.json. Do not hand-edit the lockfile.

3) Dockerfile (both stages): replace `node:lts-alpine` with `node:24-alpine`. The install line is `RUN pnpm ci` (line 7): verify whether pnpm 12.15.0 supports `pnpm ci` as a frozen-lockfile clean install (`pnpm help ci` or docs). If yes leave the line; if not, replace with `RUN pnpm install --frozen-lockfile`. Leave every other line unchanged (COPY/WORKDIR/rm/corepack/build/prune/CMD `node /app/dist/index.js`).

4) tsconfig.json: change ONLY if TypeScript 7 requires it to compile. If changed, keep `module: nodenext`, `rootDir ./src`, `outDir ./dist`, `types: ["node"]`, and CommonJS output (dist/index.js must still run via `node /app/dist/index.js`).

5) src/index.ts: change ONLY if the new axios 1.20.0 / TypeScript 7 types force a compile fix. No behavioural changes.

Do NOT change: .github/workflows/docker-image.yml, README.md, .github/copilot-instructions.md, .dockerignore, .gitignore.

Self-check before committing: `pnpm install --frozen-lockfile` succeeds with pnpm 12.15.0; `pnpm run build` succeeds and produces dist/index.js which matches the new `main`; `node --check dist/index.js` passes; the compiled output requires axios via `require("axios")` (CommonJS); `pnpm audit --prod` reports no known vulnerabilities (if one remains in a transitive dep with no fixed release, note it for the summary — do NOT add pnpm overrides). If docker is available run `docker build .`; otherwise note that in the summary. Then commit with git add -A && git commit and call report_code_changes.
**Testing**: Build and verify in drone-portainer: run `pnpm install --frozen-lockfile` with pnpm 12.15.0 (clean state first), `pnpm run build` with TypeScript 7, then verify: (1) dist/index.js exists and matches package.json `main`; (2) `node --check dist/index.js` passes; (3) the compiled output loads axios via `require("axios")` i.e. CommonJS; (4) `pnpm audit --prod` reports no known vulnerabilities (document any remaining unfixed transitive advisory instead of adding overrides); (5) grep the Dockerfile for `lts-alpine` — zero matches, both stages pin node:24-alpine, and the install command is one pnpm 12.15.0 actually supports; (6) docker build if docker is available, otherwise state unavailability. This repo has no test suite and none is required — this phase is validation of the acceptance criteria. Report findings via report_test_results.
**Review**: Review in the drone-portainer repository. Focus on the workspace diff (+/- lines): package.json field changes must be exactly main -> dist/index.js, axios ^1.20.0, typescript ^7.0.2, @types/node major 24, and a Corepack-generated packageManager hash (verify the sha512 matches what `corepack use pnpm@12.15.0` produces — a hand-written/truncated hash is a defect); version must remain 1.0.0; no other fields changed. pnpm-lock.yaml must be regenerated by pnpm 12.15.0 (lockfileVersion consistent with pnpm 12) and not hand-edited. Dockerfile: both stages node:24-alpine, install line valid for pnpm 12.15.0, everything else byte-identical. tsconfig.json / src/index.ts only changed if genuinely required to compile, with no behavioural changes. Strict exclusions: .github/workflows/docker-image.yml, README.md, .github/copilot-instructions.md, .dockerignore, .gitignore must be untouched. Use the testing phase results to verify build/audit checks pass — do not reject because you cannot run tests yourself. Call report_review_verdict.

### coder (narrative)

The container had no node/npm/pnpm/corepack and the repo has no setup skill, so I installed Node 24 LTS into /tmp (outside the repo) to run preflight and validation; preflight confirmed all four target releases exist and `pnpm audit --prod` on the old lockfile showed 25 advisories. Edits were small and surgical: package.json (main, axios, typescript, @types/node, packageManager generated by `corepack use pnpm@12.15.0` so the sha512 is real), the regenerated lockfile, and both Dockerfile stages pinned to node:24-alpine. tsconfig.json and src/index.ts needed no changes — TypeScript 7 compiled cleanly and still emits CommonJS with require("axios"). Two things surprised me and needed investigation rather than acceptance: after the axios bump the audit still flagged 2 high advisories in transitive follow-redirects 1.16.0, so I re-resolved it to 1.16.1 (which axios's own ^1.16.0 range allows — no overrides added); and the regenerated lockfile appeared to contain duplicate top-level YAML keys, which I proved was pnpm 12's legitimate two-document format by reproducing it in a clean-room /tmp project, not corruption. I kept `RUN pnpm ci` because `pnpm help ci` on 12.15.0 documents it as a clean install with a frozen lockfile, and verified it runs green alongside `pnpm install --frozen-lockfile`. Docker is unavailable in this environment, so `docker build .` could not be run.

### tester (narrative)

The environment had no node, npm, pnpm, corepack or docker, so I bootstrapped Node 24.22.1 into /opt/node and activated pnpm 12.15.0 through Corepack before starting validation. I then reproduced the full Dockerfile command sequence locally (corepack enable pnpm -> pnpm ci -> pnpm run build -> pnpm prune --prod -> node dist/index.js), which let me validate the builder stage even though `docker build .` was impossible.

The main difficulty was proving the acceptance criteria are load-bearing rather than accidentally true. I did this with three negative controls: mutating axios to ^1.21.0 made `pnpm install --frozen-lockfile` fail with ERR_PNPM_OUTDATED_LOCKFILE; installing the merge-base lockfile failed with "Cannot update packageManagerDependencies"; and a bogus sha512 in `packageManager` made Corepack abort with a hash mismatch, which also confirmed the committed hash matches the real npm tarball digest. I additionally reproduced the two-document pnpm-12 lockfile shape in a clean-room project to confirm it is normal, not corruption, and regenerated the lockfile to confirm it is byte-identical to the committed one.

The most interesting investigation was TypeScript 7's new native Go compiler. Because the platform package declares no `libc` field and there is no musl variant, Alpine would receive the glibc-flavoured binary; I verified via ELF header inspection (no PT_INTERP, zero GLIBC_ references) and a musl-simulated install that the binary is fully static and works, so this is a latent risk rather than a current failure. I filed it as a low-severity issue and left the production code untouched, as this phase is read-only validation.

### reviewer (narrative)

I reviewed the full goal, coding report, testing report, and complete merge-base diff, then used read-only assertions to verify that only authorized package values and Docker image tags changed. The tester’s byte-identical lockfile regeneration and independently verified Corepack digest resolved the potentially confusing pnpm 12 multi-document format and integrity concerns. Every acceptance criterion was supported by the diff or recorded validation; Docker execution was unavailable as explicitly allowed, and no changes or new issues were needed.

### Brain Summary (Final)

drone-portainer's dependencies were modernized in a single clean iteration touching exactly three files: package.json (axios ^1.17.0 → ^1.20.0 clearing all 25 known advisories, typescript ^6.0.3 → ^7.0.2, @types/node to ^24.19.2 on major 24, Corepack-generated packageManager hash for pnpm@12.15.0, and `main` fixed from the nonexistent dist/server.js to the actually emitted dist/index.js), pnpm-lock.yaml (regenerated by pnpm 12.15.0, with transitive follow-redirects re-resolved to 1.16.1 so `pnpm audit --prod` is clean with no overrides), and the Dockerfile (both stages pinned from floating node:lts-alpine to node:24-alpine; `pnpm ci` was kept since pnpm 12.15.0 supports it as a frozen-lockfile clean install). tsconfig.json and src/index.ts needed no changes — TypeScript 7 compiled the project as-is, producing CommonJS output still runnable via `node /app/dist/index.js`. The tester independently proved the lockfile was pnpm-generated (byte-identical reproduction, negative controls on both the lockfile and the packageManager hash verified against the real registry tarball digest) and substituted a verbatim replay of the Dockerfile builder stage since no container runtime was available; version stayed 1.0.0 as required. Future goals should note: TypeScript 7 ships a native compiler via platform optionalDependencies with no musl variant (currently harmless — the Go binary is statically linked — but flagged as a low-severity issue), Node 26 remains a deliberate later move, and README/changelog updates are still owed by the separate release-metadata goal.