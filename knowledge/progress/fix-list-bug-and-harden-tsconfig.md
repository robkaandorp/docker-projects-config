---
title: Progress: fix-list-bug-and-harden-tsconfig
type: scratch
status: draft
author: composer
tags: []
links: []
created: 2026-09-28
updated: 2026-09-28
---

# Progress: fix-list-bug-and-harden-tsconfig

## Iteration 1

### Brain Plan

Phases: Coding → DocWriting → Testing → Review → Merging

Reasoning: First iteration with no prior feedback, so no Improve phase. The goal bundles a code change (List bug fix, strict tsconfig + @types/node/lockfile regen, RBD_CONF_CLUSTER/RBD_CONF_KEYRING_USER implementation) with an explicitly requested README update, so DocWriting runs after Coding in the same content block, followed by one Testing round (R2/R7), then a single Review and Merging. Coding is escalated to premium because it spans multiple files with subtle backwards-compatibility constraints (identical rbd argument arrays when env vars unset) and a lockfile-regeneration step that must not be hand-edited.

**Coding**: Implement all code and config changes in the docker-volume-rbd repo. Work through them as four coherent parts:

1) VolumeDriver.List bug (src/server.ts, around line 255): the returned object uses `Name: name`, where `name` is undefined in that scope and only compiles due to the DOM global `name`. Change it to `Name: info.image`.

2) tsconfig.json hardening: keep module=nodenext, target=esnext, rootDir=./src, outDir=./dist and add "strict": true, "lib": ["es2023"] (removing the implicit DOM lib so browser globals like `name` no longer type-check) and "types": ["node"]. Fix any compile errors strict mode reveals — e.g. in src/rbd.ts isMapped, replace the `any[]` for the parsed `rbd showmapped --format json` output with a properly typed structure (an array of entries with pool, name, device fields, or an index-signature object if rbd emits object-keyed JSON — match what the code actually parses) — but do NOT change runtime behaviour beyond the fixes described in this goal.

3) RBD_CONF_CLUSTER / RBD_CONF_KEYRING_USER implementation (src/server.ts + src/rbd.ts), fully backwards compatible:
- In server.ts read both env vars WITHOUT defaults, so the value is undefined when unset or empty. Remove the two `// ToDo: Not utilised currently` comments there.
- In src/rbd.ts make `cluster` and `user` optional in the constructor options type and remove the `// ToDo` comment at the top of the class.
- Add a private helper in Rbd that returns the common rbd args: include `--cluster <cluster>` only when cluster is set and `--id <user>` only when user is set. Spread it into every rbd invocation: showmapped (isMapped), map, unmap, list, create, and trash move (remove). Do NOT add these args to mkfs, mount or umount.
- When both env vars are unset, every rbd command line must be byte-for-byte identical to today's (pool-only args, same ordering relative to existing flags is acceptable as long as the arg array is unchanged when unset).

4) package.json + pnpm-lock.yaml: add @types/node as an explicit devDependency, choosing a major matching Node.js LTS (what the Dockerfile installs via nodesource setup_lts.x — currently Node 22). Then regenerate pnpm-lock.yaml by running pnpm itself — NEVER hand-edit the lockfile. Use the pnpm version pinned by the packageManager field (pnpm@12.4.2), e.g. via `corepack enable pnpm` (as the Dockerfile does) or `npx pnpm@12.4.2 install`. Do not change the packageManager field. The lockfile legitimately contains two YAML documents (pnpm's own packageManagerDependencies entry, then the project deps) — keep that structure. Verify `pnpm install --frozen-lockfile` succeeds from a clean state, then verify `pnpm run build` passes with strict mode on and no DOM lib.

5) Dockerfile: rewrite ONLY the comment at lines ~3-5. The current comment wrongly claims Ubuntu's native ceph-common 19.2 (Squid) is used because 20.x breaks `rbd map` on kernel 5.10. Reality: ceph-common is installed from the official download.ceph.com debian-tentacle repo (Ceph Tentacle 20.2), and the swarm hosts have been upgraded to Tentacle 20.2. Write a comment stating that. Do NOT touch the RUN instructions or anything else in the Dockerfile.

Commit with git add -A && git commit. Files NOT to change: src/mountPointEntry.ts, config.json, entrypoint.sh, build.sh, .github/workflows/docker-image.yml, .github/copilot-instructions.md, README.md (README is handled by a separate docwriter phase).
**DocWriting**: Update ONLY README.md in the docker-volume-rbd repo, only in the options list section (the lines around 14-24). Remove the two 'not yet implemented' notes under RBD_CONF_CLUSTER and RBD_CONF_KEYRING_USER, and replace the 'default: ceph' / 'default: admin' entries: both options are unset by default — when unset the rbd CLI uses its own defaults (cluster `ceph`, user `admin`). Document that RBD_CONF_KEYRING_USER is the Ceph user name WITHOUT the `client.` prefix — e.g. setting `customuser` makes Ceph look for `/etc/ceph/<cluster>.client.customuser.keyring`. Make no other README changes. Do not run git diff; use your workspace context to see your changes. Build to verify nothing is broken, then call report_doc_changes.
**Testing**: Validate the coder's changes. Build the project (pnpm/tsc via build and test skills) and confirm: (a) `pnpm run build` succeeds with strict mode on and no DOM lib — this also proves the List bug fix compiles for the right reason now; (b) from a clean state (no node_modules), `pnpm install --frozen-lockfile` with pnpm 12.4.2 succeeds, proving package.json and pnpm-lock.yaml agree — the lockfile must still contain its two YAML documents; (c) inspect the compiled/typed rbd call sites in src/rbd.ts: with RBD_CONF_CLUSTER and RBD_CONF_KEYRING_USER unset the argument arrays for showmapped, map, unmap, list, create and trash move are identical to the previous ones, and when set, `--cluster`/`--id` appear in all six rbd calls and in none of mkfs, mount, umount; (d) server.ts reads both env vars as undefined-when-unset and no longer has the ToDo comments. There is no test framework yet (added in a later goal), so do not invent one; verify via build/type-check and code inspection. If the changed behaviour lacks any authorable tests within these constraints, note it in your report rather than failing the phase. Call report_test_results.
**Review**: Review the complete change set in docker-volume-rbd against the acceptance criteria using your workspace diff commands, focusing on +/- lines: (1) server.ts List handler returns `Name: info.image`; (2) tsconfig has strict:true, lib ["es2023"] (no DOM), types ["node"], original settings preserved; (3) @types/node added as devDependency with an LTS-matching major, packageManager untouched, and pnpm-lock.yaml regenerated by pnpm (two YAML documents preserved) — flag any hand-edited lockfile as a rejection; (4) Dockerfile comment rewritten to describe the debian-tentacle 20.2 repo and upgraded hosts, RUN instructions untouched; (5) Rbd helper emits --cluster/--id only when set, applied to all six rbd invocations and none of mkfs/mount/umount, and no-arg arrays identical when both env vars are unset; (6) ToDo comments removed; (7) README options section updated as specified with no other changes; (8) none of the excluded files (src/mountPointEntry.ts, config.json, entrypoint.sh, build.sh, .github/workflows/docker-image.yml, .github/copilot-instructions.md) changed. Use the testing phase results to verify all tests/build pass — do NOT reject because you cannot run tests yourself. Call report_review_verdict.
**Merging**: Merge the feature branch containing the List fix, strict tsconfig, @types/node + regenerated lockfile, Dockerfile comment correction, RBD_CONF_CLUSTER/RBD_CONF_KEYRING_USER implementation, and README update. Verify the full change set matches the goal's Files to change list and that nothing outside it was touched.
