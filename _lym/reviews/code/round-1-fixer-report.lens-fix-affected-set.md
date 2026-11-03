```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the set searches for F1, F2, F4 and F5 and checked that the fix covers every member.

**Findings**

1. **MEDIUM** — `internal/loomcli/driftwiring_integration_test.go` (new test) against `internal/loomcli/driftwiring.go:143`.
   - **Summary:** The F2 fix leaves one member of the call-site set unpinned. The loomcli `StartCapture` seam passes `AnchorDir: location.AnchorPath()` to the real spawn.
   - **Evidence:** `TestDriftDeps_RunInTheAnchoredDirectory` calls only `deps.RunCapture` and `deps.Compare`. Reverting `driftwiring.go:143` to `WorktreePath()` fails no test.
   - **Why the other call sites hold:** The driftcli `capture` path is covered by the integration test, because a child spawned outside the anchor cannot resolve its worktree. The loomcli seam is not.
   - **Why it matters:** The review's F2 fix said that reverting any of these call sites must fail a test. The fixer knows `os.Executable` blocks testing it here, and a plain test cannot spawn the test binary as `lym`.

**Checked, no gap found**

- **F2, other call sites:**
  - `grep` for `WorktreePath()` and `.root()` in `internal/driftcli` and `internal/loomcli/driftwiring.go` shows the remaining worktree-root uses are intentional.
  - `cli.go:409` uses `root()` for `CaptureJobName`. `driftwiring.go:111`, `driftwiring.go:128` and `driftwiring.go:182` use it for `WorktreeRoot` in Deps and the seat tables.
  - All four driftcli request sites and the loomcli `RunCapture`, `Compare` and `StartCapture` sites use `AnchorPath()` or `w.anchor()`.
- **F5, start-error sources:**
  - `startFailure` is the only producer of `childNoRecordError`, at `spawn.go:128` and `spawn.go:133`, and both callers of `StartCapture` pass its error through unwrapped.
  - `errors.As` therefore works.
  - The `cmd.Start` failure at `spawn.go:97` stays a plain wrapped error, so it is returned. That matches the review's "environment fault" intent.
- **F1, capture-record writers:**
  - `capture.go:77` is the only non-empty `CaptureRecord` constructor. Later writes reuse `rec`, so `Skip` persists through the `OnSlot`, `OnStart` and `failCapture` writes.
  - `RecordScope` is the only place that replaces `Scope` and `Skip` (`verbs.go:96`).
  - The main-capture skip sources are `scope.Skip` in both `cli.go:422` and `driftwiring.go:167`, which matches what is stored in the record.
- **F4, holder sites:** The other `Holder` constructors (`planglyph`, `gatecli`, `cardverify`) are unaffected by the doc-comment change.

**Further tool calls**
1. `grep` for `WorktreePath()` and `.root()` in driftcli and `driftwiring.go`, plus `CaptureRecord{` and `StartCapture` and `startFailure` sites. My context lacked a full-tree view of these sets.
2. `grep` for `.Skip` and `Skip =` across the drift packages, and for `gateslot.Holder` constructors. My context lacked the consumer and writer sets for F1 and F4.
