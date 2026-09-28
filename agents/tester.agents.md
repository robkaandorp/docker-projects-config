# Tester guidance

- Before testing, reconcile the changed-file set with the goal’s expected files. Investigate omissions and distinguish deliberate out-of-scope files from missed work.
- When preserving external process command lines, test the built/public-facing code while intercepting process-launch calls. Compare complete executable and argument vectors across relevant configuration combinations, including unset and empty values, against baseline behavior; account for nested internal calls instead of assuming an idealized call list.
- For compiler or tool-configuration fixes, use a small negative-control input that reproduces the former bug and confirm it is rejected with the corrected configuration. This demonstrates that the fix is load-bearing.
- When dependency metadata changes, verify that a clean frozen-lockfile install succeeds using the repository’s declared package manager and version.
