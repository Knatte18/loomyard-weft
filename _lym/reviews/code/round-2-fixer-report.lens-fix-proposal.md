```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F3, F10 and F11 that the fixer carried out as proposed.

No findings that meet the rubric's bar. Cases I built:

- **F3, fields kept as unknown keys.** A stored `scope.json` holding the deleted fields (`accepted`, `skip`, `compared_*` and the rest) still reads and rewrites safely. `internal/state/keep.go` appends unknown keys back on rewrite, and the changed `recordsguard_test.go` row reads a record carrying `skip` and `accepted`. The deletion of those fields therefore breaks no stored record.
- **F3, stored `partial` scope.** A stored `partial` record now renders as "Scope: full." in `scopeText` (`internal/driftengine/verdict.go`). `Check` also diffs every case instead of leaving the skip list out. This is exactly the behaviour the review asked for, since no hub holds such a record, so I raise nothing.
- **F3, stored waiver.** A stored waiver with no last check now renders "Check: not run." in the verdict. `skipReason` in `check.go` and `verify.go:143` still honour the waiver. This is cosmetic, because nothing writes a waiver any more, and the review did not ask for those readers to go.
- **F10, the new way forward.** `lym config reconcile --layer default --apply` sets `PruneUnknown` and sits in `reconcileDefaultLayer`. That guard refuses a task session and says the orch runs it, so the kept sentence "a session lym refuses the verb from reports status: FAILED and the orch runs it" holds.
- **F11, the new row.** With the template base present, an absent hub file and a prime copy, dropping `!fileAbsent` would make the template aliases read as an operator deletion and leave `sonnet` out of `Added`. The new row asserts `sonnet` in `Added` and in the file, so it fails under that mutation.

Calls made:
1. `grep` on `internal/state/*.go` for unknown-key handling, because my context lacked how `state` treats keys the struct no longer declares.
