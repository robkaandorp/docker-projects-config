---
title: Review: update-copilot-instructions
type: scratch
status: draft
author: reviewer
tags: []
links: []
created: 2026-09-28
updated: 2026-09-28
---

# Review: update-copilot-instructions

## Verdict: NeedsChanges

## Issues

### [MAJOR] The claim that a conflicting device at the mountpoint always causes Mount or Unmount to return an Err and change nothing is too broad. In src/app.ts, that check applies when the volume is untracked. A tracked Mount reuses its table entry without checking the mounted device, and a tracked Unmount does not perform that conflict check. Limit the instruction to untracked-volume paths.

## Verified
- ✅ .github/copilot-instructions.md and all requested source, test, build, configuration, skill, and workflow files exists
- ✅ parseConfig, Rbd, createApp, RbdInterface, createVolumeLock, withVolumeLock, cleanupBestEffort, CommandRunner, FileSystem, getMountedDevice, MountPointEntry exists
- ✅ Node pin, build and test scripts, TypeScript settings, Docker stages, VERSION, and CI build/publish jobs exists

## Recommendation
Clarify that conflict detection and refusal to change anything describe untracked Mount and Unmount requests. The remaining documentation-only scope is feasible in one iteration.
