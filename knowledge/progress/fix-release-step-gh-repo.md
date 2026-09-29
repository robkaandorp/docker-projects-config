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
