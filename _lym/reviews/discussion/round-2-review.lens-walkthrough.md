```yaml
lens: walkthrough
outcome: reported
```

Task: walkthrough lens. I ran each decision's mechanism end to end on one concrete case.

**Decisions walked**

- **D1:** `cmd/testtiming -redundancy` and the gate step feed the same block sets to the shared engine. Purpose met.
- **D2:** the `internal/fabricengine` profile is filtered to blocks in the package's own files, and the module-wide column comes from the same profile. Purpose met.
- **D3:** a block shifted by an edit above it. Its file, function, text hash and ordinal stay the same, so its key holds. Purpose met.
- **D4:** a package with no test files, such as `internal/lymdirs`. Building it yields no test binary, and the record says nothing about writing a map for it. Purpose not met (finding 3).
- **D5:** a stale clean package in the prime. The dirt check, lock, HEAD and hash read, coverage build, per-test runs under the tier env, re-hash and map write chain up correctly. A package that ends with no map never stops being stale. Purpose met for a package with tests; broken for one with none (finding 3).
- **D6:** the prime dobby starts a refresh on a HEAD move and registers it under `runproc.HubOwner` as `test-map`. On the stop path, `orchcli` calls `EndRole(dir, HubOwner, dobby)`, which spares that record. Purpose of the stop claim not met (finding 1).
- **D7:** a run worktree adds tier0 `TestFoo` in `internal/x/foo_test.go`, plus an edited tier2 test in the same diff. Steps 1 to 5 chain up. In step 6 the report records hashes only for judged tests, while D8's match (which the repeat skip reuses) compares step 1's whole set. Repeat-skip purpose not met (finding 2).
- **D8:** the same diff at Code-Burler-Review. Running step 1 on HEAD returns `TestFoo` and the tier2 test. The report holds `TestFoo` only, so the match fails and the marker says no current report exists. Purpose not met (finding 2).
- **D9:** prose only. Nothing to execute; review check.

**Findings**

1. **BLOCKING · design · D6 ("A dobby stop ends the child with the dobby, since `runproc.EndRole` ends every descendant…") and the D6 criterion "a dobby stop ends a running refresh".**
   The stated mechanism does not end the child.
   - The prime dobby is registered under `runproc.HubOwner` with role `dobby` (`internal/beamcli/doc.go:353`, `internal/beamcli/dobbydeps.go:77`). Its stop calls `runproc.EndRole(…, owner, runproc.RoleDobby, …)` (`internal/orchcli/cli.go:276`).
   - `ender.end` lists every record of the owner and marks each record of another role as spared. The test-map child is registered under the same `HubOwner` with role `test-map`, so it is spared. The descendant walk then skips spared pids "with the processes below it" (`internal/runproc/end.go:148-166`, and the `EndRole` doc at lines 70-73: "Every record of another role, and every process below such a record, is left alone").
   - So a dobby stop leaves the refresh running, and the criterion cannot be met as designed.
   - Fix: end the `test-map` role explicitly on the dobby's stop, or register the child under another owner and state the stop route.

2. **MEDIUM · design · D7 step 6 against D8's match and D7's repeat skip.**
   The report records "per judged test its function hash". D8 matches when "step 1 of D7 run on HEAD gives the same tests with the same function hashes". Step 1's output includes every changed test, including ones steps 2 to 4 drop: `tier2`, `//testtiming:keep`, undeclared tier, past the cap, skipped, failed or timed out.
   - In a diff that touches any such test, step 1's set never equals the judged set. The marker always says no current report exists, and the D7 repeat skip, which uses D8's match, never fires, so every repeat call rebuilds and reruns.
   - Evidence: `_lym/discussion/decision-record.md:136`, `:153`, `:182`.
   - Fix: record a function hash for every test step 1 returns, judged or not, or match on step 1's set as recorded.

3. **MEDIUM · design · D4/D5 (the "it has no map" stale rule) with D6's retrigger on capped leftovers.**
   The record never says what `lym gate testmap` writes for a package that has no test files, or no `tier0`/`tier1` test.
   - `go test -c` builds no binary for a package without test files. The worktree has 8 such Go package directories, among them `internal/hibernateconfig`, `internal/lymdirs`, `internal/patternlint`, `internal/planindex` and `internal/testkit/boardkit`.
   - If no map is written, each one stays stale for good (`decision-record.md:84`), and nothing exempts it from `--max` (`:93`, `:97`).
   - Once every other package is mapped, each refresh rebuilds the first 5 of them in path order and leaves 3 as cap leftovers. D6 then starts a refresh on every tick (`:112`), and the last 3 are never reached.
   - Fix: write a map with file hashes and no tests for such a package, or a failed one, so it goes stale only when its files change; or say that such a package neither counts against `--max` nor counts as a cap leftover.

**Tool calls**

- `find` over `cmd` and `internal` for Go package directories with no `_test.go`. My context lacked how many test-free packages exist, which finding 3's counterexample needs.
