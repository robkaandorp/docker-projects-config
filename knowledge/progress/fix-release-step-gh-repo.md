---
title: Progress: fix-release-step-gh-repo
type: scratch
status: draft
author: composer
tags: []
links: []
created: 2026-09-29
updated: 2026-09-29
---

# Progress: fix-release-step-gh-repo

## Iteration 1

### Brain Plan

Phases: Coding → Testing → Review → Merging

Reasoning: First iteration, clean task with a minimal single-file change: add GH_REPO env to the release step of the publish job in .github/workflows/docker-image.yml. No documentation was requested, no prior failure/rejection exists, and there is no missing test coverage to author (the repo's node:test suite doesn't cover CI workflow YAML and the goal explicitly limits changes to that one file), so a single coding → testing → review → merging plan is correct.

**Coding**: Read docker-volume-rbd/.github/workflows/docker-image.yml and make exactly one change: in the 'Create the GitHub release' step (last step of the 'publish' job), add `GH_REPO: ${{ github.repository }}` to the step's env block, directly below the existing `GH_TOKEN: ${{ github.token }}` line, indented the same way. Do NOT modify the run command (`gh release create "$FULL_VERSION" --target "${{ github.sha }}" docker-volume-rbd-${FULL_VERSION}.tgz`) and do NOT add actions/checkout to the publish job — it must stay checkout-free because it runs with contents: write. Make NO other changes to the workflow: the build job, triggers, permissions, concurrency, the already-released guard, the Docker Hub steps, and step order all stay byte-identical. Do not touch any other file (src/**, package.json, pnpm-lock.yaml, tsconfig.json, Dockerfile, .dockerignore, config.json, entrypoint.sh, build.sh, VERSION, README.md, .node-version, .github/skills/**, .github/copilot-instructions.md are all out of scope). Validate the YAML parses (e.g. with a quick node/inline check that no YAML package exists, at minimum eyeball indentation consistency with the guard step at lines 80-81, which uses the same env pattern). When done, run git add -A && git commit with a concise message.
**Testing**: Run the build and test skill for the docker-volume-rbd repository (pnpm build + pnpm test). No source change is expected, but the full node:test suite must pass (108 tests) as required by the acceptance criteria. Do not attempt to run GitHub Actions — the worker environment cannot; verification of the YAML is by inspection and parsing only. Call report_test_results with the outcome.
**Review**: Review the feature-branch diff for docker-volume-rbd/.github/workflows/docker-image.yml only. Verify the acceptance criteria: (1) the 'Create the GitHub release' step's env contains both `GH_TOKEN: ${{ github.token }}` and the new `GH_REPO: ${{ github.repository }}`; (2) the release step's run command is unchanged; (3) the publish job still contains no actions/checkout and the release step is still its last step; (4) nothing else in the workflow changed — compare the diff +/- lines and confirm the only additions are the GH_REPO line (plus, if any, trivially consistent whitespace). Reject if any other file was modified or any workflow behaviour changed. Use the workspace diff commands; judge only +/- lines. The goal is a workflow-only change, so there are no new tests to author; use the testing phase results and confirm the 108 existing tests pass — do not reject because you could not run GitHub Actions yourself. Call report_review_verdict when done.

### coder (narrative)

The task was narrow and explicit: add `GH_REPO: ${{ github.repository }}` below `GH_TOKEN` in the publish job's "Create the GitHub release" step env block, with nothing else changed. That part was easy — one edit, verified with `git diff` against the merge base showing exactly `1 file changed, 1 insertion(+)`, and with `cat -A` confirming the 8-space indentation matches the guard step's env pattern byte-for-byte. Since no YAML parser exists in the repo (and the lockfile may not be regenerated), I relied on that structural comparison instead of adding tooling. Bonus: gh 2.101.0 was available, so I reproduced the real failure in a scratch non-git directory — `gh release list` failed with the exact "fatal: not a git repository" error, while the same command with GH_REPO set succeeded, plus a read-only `gh release view v20.2.26` confirming the target resolves to robkaandorp/docker-volume-rbd. For the regression self-check I used the setup-node skill, did a frozen-lockfile install, `pnpm run build`, and `pnpm test`, which passed 108/108. The only stumble was a shell-quoting mistake in my first `git commit -m` (an unbalanced quote broke the heredoc-free one-liner); I retried with a message file and committed cleanly. Remaining uncertainty is that the write path (`gh release create` with a write token) can only be truly exercised in CI.

### tester (narrative)

This was a verification-only iteration on a single-line CI workflow change (GH_REPO env added to the gh release create step). I confirmed via git diff that only .github/workflows/docker-image.yml changed with no type-level impact, then ran the standard toolchain: the setup-node skill had to be used because node was not on PATH (it reused a cached Node 24 install, so setup was fast), a fresh frozen-lockfile pnpm install, tsc build, and the full node:test suite — all 108 tests passed on the first attempt with exit 0. No test authoring was needed since this is pure workflow YAML in scope-restricted code, and the working tree stayed clean, so there was nothing to commit before reporting. The only mild friction was remembering that PATH updates don't persist between shell calls, so I re-exported the node bin dir in every command that needed it.

### reviewer (narrative)

I compared the merge-base diff and found exactly one added GH_REPO entry in the release step's env block, then read the workflow to confirm placement, indentation, unchanged command, and checkout-free publish job. I reviewed the independent tester report showing a successful build and all 108 tests passing. PyYAML was not available locally, so I checked the single-line YAML edit by inspection; there were no other changed files or behavioral changes to assess.

### Brain Summary (Final)

The "Create the GitHub release" step of the publish job in .github/workflows/docker-image.yml received a one-line fix: `GH_REPO: ${{ github.repository }}` was added to the step's env next to GH_TOKEN, so gh resolves the target repository explicitly instead of via git remote inference, which was the root cause of the v20.2-r1 release failure (`failed to run git: fatal: not a git repository`) in the deliberately checkout-free publish job. Everything else in the workflow — the build job, triggers, permissions, concurrency, the already-released guard, the Docker Hub steps, and step order — is byte-identical, and the publish job still contains no actions/checkout so it never runs repository code with write permissions. The fix was verified at commit 7a08599 by inspection plus a read-only gh 2.101.0 demonstration in a scratch non-git directory (without GH_REPO: "not a git repository"; with it: successful release listing), and the 108-test node:test suite still passes. For future goals: the actual `gh release create` with a write token can only be validated on the next real master push in CI, so the owner should confirm the release step on the next release run.