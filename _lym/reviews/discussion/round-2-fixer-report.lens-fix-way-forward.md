```yaml
lens: fix-way-forward
outcome: reported
```

fix-way-forward lens: I checked every way forward, refusal, next step and criterion that the fix added.

**LOW** — D9, the changed sentence "Only a `--upkeep` call does the one-pass-at-a-time check, which reads every `runproc.RoleTestMap` record under its worktree's owner; in a task, a hand call therefore holds off a pass until it ends."
Summary: when a hand rebuild holds off a pass in a task, the refusal names the wrong holder, and the record never says that its text changes.
Evidence:
- `registerUpkeepPass` refuses with "the upkeep pass %d is running for the hub; way forward: wait for it to end" (`internal/gatecli/testmap.go:345`).
- After the fix, the registration it finds in a task can be a hand named-package rebuild under the task's slug, not an upkeep pass of the hub.
- Waiting still makes progress, so the way forward can be carried out. But the refusal misnames the holder and the scope, and `PATTERN-refusal-way-forward` (Constraints) covers D9's refusals.
- D9 should say that the refusal names the registered `lym gate testmap` call of the worktree, not "the upkeep pass for the hub".

No other findings:
- **D4 "its commit waits for the pair's next push":** the push it waits for exists. A held-back verdict pauses the entry, and the commit goes with the next push after a resume. The teardown's archive push of the weft branch also carries it.
- **D9 teardown bound:** it holds. `commitPendingRecords` (`internal/fabricengine/remove.go:792-845`) stages the record pathspec entries in the sibling weft worktree during `Remove`, and the owned processes are ended before that.
- **D5 pull commit "under fabric's records write lock":** it can be reached without self-deadlock. The pull takes the weft write lock only later, in `lockWeftForReplay` (`internal/fabricengine/pull.go:426`, `:502-514`), so a commit before the weft moves can take it first.
- **D11 lock-skip "imported by a later call":** it makes progress. Any later prime call re-runs the import for a package that still has no map.
- **New D4, D5 and D9 criteria:** each starting state can be built: an entry the reconcile reads as ended, a prime weft dirty only under `_lym/testcover/` with origin moved, and a hand call with and without packages named.

Tool calls:
- Read `internal/fabricengine/remove.go:790-845`: my context held only the grep line for `commitPendingRecords`, not its pathspec and lock behaviour.
- Grepped `internal/fabricengine/pull.go` for lock acquisition: my context lacked which lock the weft pull holds before replay.
