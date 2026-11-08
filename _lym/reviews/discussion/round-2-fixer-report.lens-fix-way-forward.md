```yaml
lens: fix-way-forward
outcome: reported
```

I checked whether every refusal, error, next step and criterion that the fix added can be reached from the state it names and leads somewhere useful.

**[MEDIUM] A package whose coverage build failed has no route back to a real map**
- Changed text: D5, "A package whose coverage build fails … gets a map recording its Go file hashes and no tests, with the build failure as its cause … stale again only when its files change or its map passes `test_map_max_age_days`". Also D5, "a build past it is ended with its process group like a test and counts as failed". Also the new D5 criterion, "neither is rebuilt on the next call while its files are unchanged".
- Issue: once a build fails, nothing reports the failure and nothing points to a retry.
  - A failure can come from outside the package: a broken dependency, or a transient overrun of `redundancy_build_timeout_min`.
  - When that cause is fixed, the package's own files are unchanged, so it is not stale. Neither `--max` nor D6's prime-HEAD or capped-leftover triggers rebuild it until `test_map_max_age_days` passes, 7 days by default.
  - The verb's exit lists only skipped packages and the count the cap left unbuilt. A failed-build package is neither, so its failure is never shown.
  - In D7, the package has a map, so the "a package with no map is reported as not judged" line and D7's no-map repeat retry never fire for it. The map holds no tests, so the package's tests are judged with no map covers, and no line tells Code-Review that its map is a failed build.
  - The only way out is an explicit `lym gate testmap <package>`, and no output tells anyone to run it.
- Evidence: D5's exit sentence, "It exits listing each skipped package with its cause, … apart from the count of stale packages the `--max` cap left unbuilt". D7 step 6, "A package with no map is reported as not judged". D7's repeat-skip exceptions are "an error line, a package with no map or a test past the budget".

**[LOW] D7 step 1 has no named way to list a package's test files at the merge base**
- Changed text: D7 step 1, "the step reads the package's `_test.go` files at the merge base through `fabricengine.CodeFileAt` (new), which reads a file of the warp checkout at a revision".
- Issue: `CodeFileAt` reads one named file, but the rename lookup needs the list of `_test.go` files the directory held at the merge base, and no route for that listing is named.
- The record also gives no outcome for a package directory absent at the merge base. If that falls under "an unreadable merge base", it becomes an error line. Every repeat call then reruns, because a report holding an error line never skips the step.
- Evidence: D7 step 1 as changed. D7 lists "an unreadable merge base" among the error lines, and "A report that holds an error line … never skips the step".

Outside my lens: `internal/fabriccli/anchordeps.go:258` ends the hub owner's processes through `runproc.EndSparing`, so an anchor also ends a `test-map` child. The next dobby's first tick restarts the refresh, so no step is stranded.

Tool calls:
- grep for `RoleDobby` across non-test Go files, to find every dobby stop route the new F2 sentence must hold under. My context had only `internal/orchcli/cli.go`.
- grep of `internal/fabriccli/anchordeps.go` for `EndRole`, `EndProcess`, `EndSparing` and `func`, to see how the anchor ends dobbies. Not in my context.
- Read of `internal/fabriccli/anchordeps.go:240-269`, to confirm what the anchor's `EndDobbies` ends. Not in my context.
