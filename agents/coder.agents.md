# Coder guidance

- Before completing implementation, reconcile the changed-file set with the goal’s expected files. Investigate omissions and explain deliberate additions or exclusions rather than treating diff size as proof of completeness.
- Use the package manager and version declared by the repository for dependency operations. Never hand-edit lockfiles; validate dependency metadata with a clean frozen-lockfile install when applicable.
- If a required runtime or tool (e.g. `node`) is missing or the wrong version, look for a setup skill under the repository's `.github/skills/` and use it before downloading anything ad hoc. Never commit installed toolchains.
