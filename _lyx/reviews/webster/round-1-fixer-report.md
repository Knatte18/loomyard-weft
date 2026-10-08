# Webster-Burler round 1 — fixer report

All four findings were confirmed against the code and fixed, one commit each, plus a test-budget commit for F1's new test.
After the fixes, `go vet` (default and `integration`), `go test ./...`, `go test -tags integration ./...` and `lyx loom lint-comments --base c0bf19c54` all pass.

## Fixed

### F1 — new error paths with no way forward and no spec row

- `internal/webstercli/reset.go`: once the reset has moved the branch, archived the record and synced, a failed `UncommittedPaths` read is no longer a refusal.
  The verb now returns the ok envelope without `uncommitted` and adds a warning: `the uncommitted paths could not be read: <err>; `git status` in the task worktree lists them`.
  A caller can therefore tell the reset happened, and nothing points it at a re-run that would refuse on the archived record.
  The long help and the `websterengine` package doc (`doc.go`, reset envelope paragraph) state the new behavior.
  A new `TestResetCmd` step reaches it: a standalone archive-only reset over a fake `websterengine.Git` whose `DirtyPaths` fails.
- `internal/websterengine/runlevel.go`: `Run` wraps the `overviewFrameHash` error with `way forward: transient, re-run `lyx webster run``.
- `internal/websterengine/fingerprint.go`: `restampBaseline` wraps the same error with `way forward: transient, re-run the verb`.
- `contracts/specs/refusal-spec.md`: new `overview frame unreadable` transient row.
- `internal/websterengine/rebaseline_test.go`: new `TestRebaseline_OverviewFrameUnreadable` reaches the row through both `Rebaseline` paths: the index-only check when the state has plan-file hashes, and the restamp when it has none.
  The overview loses its Card Index after the plan is parsed, which simulates the file changing between the two reads.
- `cmd/lyx/testdata/test-budget.yaml`: `internal/websterengine` raised from 189 to 190 for that test, in its own commit (`84ddbadcf`).

Commits: `57deb3dee`, `84ddbadcf`.

### F2 — transient refusals without a spec row

`contracts/specs/refusal-spec.md` gains a `transient` row for each:

- `Merriam start base unreadable`;
- `recover-batch: uncommitted paths unreadable`.

The rebaseline overview read is covered by F1's `overview frame unreadable` row, and `TestRebaseline_OverviewFrameUnreadable` reaches it.
Commit: `9ad02ea04`.

### F3 — `MigrateConfig` misplacing or corrupting `orientation`

`internal/batcher/config.go` now takes the lowest index across the drops, the one the second drop leaves valid.
A new `TestMigrateConfig` row (`startup_context` first, a trailing `master_base`) failed first with a slice-bounds panic and passes now.
Commit: `0994db990`.

### F4 — dash pair in run's long help

`internal/webstercli/run.go`: `for an edit it accepts -- --fresh archives` became `for an edit it accepts; --fresh archives`.
The `--` was the only problem, and the parenthetical it sits in runs on to `by hand)`, so a semicolon fixes the flag-pair reading and leaves the sentence structure as it was.
Commit: `37052fdf8`.

## Disputed

None.

## Deferred with reason

- F1 asks for a test that reaches each new path. No test reaches the `Run` wrap of `overviewFrameHash`.
  `Run` calls `planparser.ParsePlan` on the same directory first, and that call reads `00-overview.md` and refuses a missing Card Index.
  The wrap can only fire if the file changes between the two reads in one call, and `Run` has no seam to inject that change.
  The row is reached through `Rebaseline` instead, which shares the same `overviewFrameHash` failure.
