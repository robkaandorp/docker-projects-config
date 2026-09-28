# Tester guidance

- Before testing, reconcile the changed-file set with the goal’s expected files. Investigate omissions and distinguish deliberate out-of-scope files from missed work.
- When preserving external process command lines, test the built/public-facing code while intercepting process-launch calls. Compare complete executable and argument vectors across relevant configuration combinations, including unset and empty values, against baseline behavior; account for nested internal calls instead of assuming an idealized call list.
- For compiler or tool-configuration fixes, use a small negative-control input that reproduces the former bug and confirm it is rejected with the corrected configuration. This demonstrates that the fix is load-bearing.
- When dependency metadata changes, verify that a clean frozen-lockfile install succeeds using the repository’s declared package manager and version.
- If a required runtime or tool (e.g. `node`) is missing or the wrong version, look for a setup skill under the repository's `.github/skills/` and use it before downloading anything ad hoc.
- When a setup skill prints environment exports, reapply them in every fresh shell; `PATH` changes do not persist.
- When fixture content relies on escapes or newline semantics, verify its bytes with `od -c` before running the test; for example, `printf '%s'` writes `\n` literally. Diagnose fixture or harness errors before attributing a failure to the code under test.
- When testing input validation, include malformed variants with internal whitespace, embedded separators, and values empty after trimming, and state which negative cases ran. For tooling/environment criteria in a repository without a test framework, focused functional runs are authoritative; do not introduce a framework solely for validation.
