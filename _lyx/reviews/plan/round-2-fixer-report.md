# Round 2 fixer report — plan

## Changes

- **F1** (MEDIUM, scope), fixed as proposed in `_lyx/plan/06-loom-fmt-verb.md`.
  The `planGoFormat` paragraph now says that only a not-exist failure of `filepath.EvalSymlinks`, matched with `errors.Is(err, fs.ErrNotExist)`, is refused as `no such file`.
  Any other `EvalSymlinks` failure is refused with its own error as the reason, and an unreadable file with its `os.ReadFile` error.
  `TestFmtCmd` gains a row: a directory named `dir.go` inside the worktree, named beside a misformatted file, refuses the whole call.
  `refused` names `dir.go` with its read error, and the misformatted file is left unwritten.
  The row sits inside the existing `TestFmtCmd`, so the `test-budget.yaml` bump is unchanged.

## Disputed

None.

## Deferred

None.
