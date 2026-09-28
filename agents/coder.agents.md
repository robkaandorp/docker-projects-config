# Coder guidance

- Before completing implementation, reconcile the changed-file set with the goal’s expected files. Investigate omissions and explain deliberate additions or exclusions rather than treating diff size as proof of completeness.
- Use the package manager and version declared by the repository for dependency operations. Never hand-edit lockfiles; validate dependency metadata with a clean frozen-lockfile install when applicable.
- If a required runtime or tool (e.g. `node`) is missing or the wrong version, look for a setup skill under the repository's `.github/skills/` and use it before downloading anything ad hoc. Never commit installed toolchains.
- Always finish each phase by calling its designated report tool; if the call fails, retry it. Completed work without a successful report is a failed phase, and narrative such as “done” is not a substitute.
- For values read from files or arguments, trim only leading/trailing whitespace and validate the unchanged interior against the full pattern (for example, `^[0-9]+$`); never delete whitespace globally. Self-check malformed inputs including internal spaces, embedded separators, and values empty after trimming, not only missing or plainly invalid values.
- When a setup skill prints environment exports, reapply them in every fresh shell; `PATH` changes do not persist. For tooling/environment goals in a repository without a test framework, do not add one just for the task; run and report focused functional checks, including explicit negative cases.
