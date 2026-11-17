```yaml
lens: fix-code-correctness
outcome: reported
```

Task: fix-code-correctness check of the round-1 change set (F1–F10), reading only what my context holds.

No BLOCKING findings.

1. **LOW — F2 wiring untested: `internal/gatecli/redundantmutants.go` `runMutants` and `internal/gatecli/redundancy.go` `runTest` GuardKilled handling.**
   - Only `RunAlone` itself (`runner_test.go` rows) and `JudgeMutants` (`mutantverdict_test.go`) pin the behaviour. Nothing pins what the mutation check does with a `GuardKilled` result.
   - Dropping the `guarded` bookkeeping in `runMutants` would leave every added test green. The guard-kill baseline would then read as `FailsUnmutated`. A guard-killed mutant run would be recorded as a kill.
   - The original premise was exactly that false mutant kill. It is fixed in code, but no test would fail if it regressed.

2. **LOW — F5 refusal can misfire after the command has run: `internal/gatecli/tierrun.go`, the new `run.tier.MemoryMB > 0 && runErr != nil && ... && !errors.As(runErr, &exitErr)` branch.**
   - `runMemoryCapped` returns `errors.Join(err, killErr)`.
   - Concrete input: the shell exits 0, a child is OOM-killed, so `oomKilled` is true, and `killScope` fails with EACCES. `err` is nil, so the return value is just `killErr`.
   - `killErr` is no `*exec.ExitError`, so a tier that ran and passed becomes a "did not run under its memory_mb" refusal, and `capKilled` is dropped.
   - A counter write error returned by `Wait` takes the same path.
   - Both are rare. The fix's own premise, "the scope never ran the command", is no longer what the branch tests.

3. **LOW — F2 pid reuse in `startMemoryGuard` (`internal/gatecli/memoryguard.go`).**
   - The `killed` pid set lives for the whole call.
   - `RunAlone` asks `GuardKilled(cmd.Process.Pid)` for every failing run of a long `lym gate redundant` or `lym gate testmap` call.
   - A later test process that reuses a pid the guard once killed, and fails for its own reason, is returned as `GuardKilled`. That test then goes unjudged instead of failed.
   - Linux wraps pids, so this is unlikely but possible.

4. **MEDIUM — unverified consequence of the `template.yaml` and `docs/overview.md` edits.**
   - The fixer ran the gate on `./internal/proc`, `./internal/testcover` and `./internal/gatecli` only.
   - `internal/gateslot/template.yaml` line 3 changed, and `internal/gateslot` has tests of its own. A test there may pin the template comment text.
   - `docs/overview.md` also changed.
   - Neither the `gateslot` tests nor `go build ./cmd/...` was run after the edits.
   - `cmd/testtiming/redundancy.go` calls `RunAlone` with the `Toolchain` type that gained a field, so it should still compile, but nothing shows it.
   - Run tier 0 and tier 1 over `./internal/gateslot` and `./cmd/...`.

Checked and holding:
- F1: `oomKilled` is read before `killScope`. `RunMemoryCapped` still returns `killed`. The `kill -9 $$` test fails with the old `treeKilled = true`.
- F4: the counter's `running` set matches the `go test -json` behaviour the fixer measured.
- F6: the guard-kill test fails if `memoryLines` leaves the exit or causes, or if `memoryStopped` is dropped.
- F8: `stop()` waits for the run, so both pids are recorded before the assertions.
- Callers of `guardOwnTests` (`redundant.go`, `testmap.go`) and the `counter` interface implementers are all updated.
- The `run.go` report-step guard uses `memoryGuarding` and `memoryQuit`, both set before `exe` is built.

Further tool calls: none. My context held every changed file and the review.
