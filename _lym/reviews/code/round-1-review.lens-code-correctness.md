```yaml
lens: code-correctness
outcome: reported
```

Task restated: code-correctness lens over the diff in `f98e284c..HEAD`. I traced each changed function with a concrete input and found no defect.

**Per function or branch: case run, result**

- **`fabricengine.CodeLocation`, records sibling.** I used `WorktreeName` "loomyard-weft" with the code directory present. `WeftWarpSlug` gives "loomyard", the stat passes, and `NamedWorktree(hub, "loomyard")` returns the code location with its own anchor. Correct.
- **`CodeLocation`, non-sibling and degenerate names.** A code worktree name with no `-weft` suffix comes back unchanged, and so does the bare "-weft" (empty slug). Correct.
- **`CodeLocation`, absent code directory or stat error.** A records worktree whose code directory is missing, or a non-directory, returns `l` unchanged. This is deliberate and safe. A stat error other than not-exist is also treated as absent, which is acceptable for a mapping step.
- **`CodeLocation`, `NamedWorktree` failure.** An `anchorRelOrRoot` error is returned and `resolveHub` wraps it ("resolve the code worktree of …"). Correct, with no partial state.
- **`resolveHub`, new step.** The mapping runs before `BoardLymPresent`, so the board check uses the code worktree. A cwd outside a hub is unaffected.
- **`resolveHub` consumers.** `buildRoot` returns `location.WorktreePath()` inside a hub (`prebuild.go:36`), so `request.root` is the code worktree and not the records `dir` (`run.go:79`). The other callers I saw use `location.WorktreePath()` (`cardverify.go:127`, `report.go:92`, `redundant.go:87`). Correct.
- **`testMapLocation`, from a pair's records sibling.** After the mapping, `IsPrime` runs on the pair's code worktree and returns false. `PrimeWorktreePath` then calls `PrimeName` and `NamedWorktree` and returns the prime's code path. The new row's expected message matches. Correct.
- **`testMapLocation`, from the prime's records sibling.** The mapping gives the prime's code worktree, `IsPrime` is true, and the verb acts in the prime. Correct.
- **`runInvocations`, `discoveredBefore` and delta.** The counter accumulates across invocations, so with invocation 1 reporting 3 tests and invocation 2 reporting none, the delta for 2 is 0 and the notice fires. Correct. `cmd.Stdout` is a non-file writer, so `Wait` drains its copy goroutines before `Counts()` is read, and the summary line is newline-terminated.
- **`noTestNotice`, branches.** A `go-json` counter returns "" and a delta above 0 returns "". With no targets the project falls back to `dir`. With no terms the class clause is empty. A summary of `Total: 0` or no summary at all both give delta 0 and a notice. All correct.
- **`runInvocations`, failure paths.** A non-zero exit, a cancelled context or a process-cap line returns before the notice, so no notice is printed for a failed run. Correct.
- **`runTier.notice`.** It appends to `logBuffer` and prints to stderr only after the command ended, so there is no concurrent write to the buffer. Correct.
- **`TestRunTierNoTestNotice` rows.** The command echoes a summary only for targets containing "ran", so the four rows are consistent with the code.
- **Detached-run subtest, `expireLease`.** Grace is 1m and the cap is 1s. `waitInvocations(1)` returns once the tier runs, then `expireLease(now)` sets the deadline to now−1s. The front call has already exited (`requireGoing`), so nothing rewrites the lease later. If the tier never runs, `waitInvocations` fails the test rather than hanging. The race the commit names is removed, not just widened. Correct.
- **Redundancy broken-build subtest.** The test file rewrite keeps `TestDropsHangs` out of judgment through `//testtiming:keep`, so nothing runs to the 20 s timeout. `secondUnit = time.Second` with the 20 s config gives a 20 s per-test limit. Cleanup restores `dropsTestFile` and removes `cmd/lym`. The subtest is not parallel, so the shared tree is safe. Correct.

**Findings**

1. **LOW, `design`, `internal/gatecli/tierrun.go:316,353`.** The no-test detection is only "the invocation's delta of discovered tests is 0". It cannot see a project that runs some tests while a gated class is skipped, which is the case the board entry's "which test classes it skipped and why" asks about. The line also never names `COMMONLIB_REPO`, or any other setting, as the reason. Evidence: `noTestNotice` reads only the `discovered` delta, and `vstestCounter.handle` (`counters.go:456-470`) sums the "Total" of summary lines only.
2. **LOW, `design`, `internal/gatecli/tierrun.go:93-96` with the retry path.** The notice is written into the tier's `logBuffer` and stderr per `runTier` call. A retried tier (`retryTier`) therefore repeats the notice, once per attempt. This is minor noise; I did not verify the retry path.
3. **LOW, `design`, `internal/gatecli/tierrun.go:93-96`.** `notice` writes straight into `logBuffer` while the counter may still hold an unflushed partial last line. `counter.Flush` only runs afterwards (`tierrun.go:101`). The notice can then land before the tail of the command's last line in the log. This only affects ordering in the log.

**Tool calls**

1. `Read` `tierrun.go` 70–155: lacked `runTier` and how `logBuffer` and `counter.Flush` are wired.
2. `Read` `run.go` 392–432: lacked `loadGate` and the `resolveHub` consumer.
3. `Bash` grep for `WorktreePath`, `Cwd` and `loadGate` in the other `gatecli` `resolveHub` callers: lacked the callers' use of `dir` versus `location`.
4. `Read` `run.go` 58–108: lacked where `request.root` is set after `loadGate`.
5. `Bash` grep for `buildRoot`: lacked its definition.
