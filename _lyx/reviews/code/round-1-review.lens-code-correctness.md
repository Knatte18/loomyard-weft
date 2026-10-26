```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens over the `rename-target-checks` diff, run per function and branch with a concrete input.

**Per-branch list**
- `checkRenameLeavesTest`, `NewBeam` renamed with `TestNewBeam` in Edit in the same unit: one finding on the renaming card. Correct.
- `checkRenameLeavesTest`, `TestNewBeam_Case`: matches. `TestNewBeamer`: no match, because `namesTestOf` needs `_` after the base. Correct.
- `checkRenameLeavesTest`, method `T.M` gives base `T_M`, matching `TestT_M`. Correct.
- `checkRenameLeavesTest`, test already in `renamed[ref]`: skipped. Correct.
- `checkRenameLeavesTest`, csharp or none plan: returns empty. Correct.
- `checkRenameLeavesTest`, file-rename pair, self glyph, or `len(old.Owner) > 1`: skipped. Correct.
- `checkRenameLeavesTest`, test in an external test package (`package foo_test`): not matched. **Wrong; see finding.**
- `RemovedTargets`, deleted symbol declared by the batch's own Delete target or pair old side, or listed in the report's renames: no finding. Correct.
- `RemovedTargets`, deleted symbol with a later card's target or `Uses:` ref: one blocking finding per card. Correct, since `targetCards` indexes `Targets`, `Uses` and both pair sides.
- `RemovedTargets`, symbol quarry renamed exactly: absent from `Deleted`, so no finding. Correct.
- `RemovedTargets`, empty `Deleted` or empty `pending`: returns nil. Correct.
- `postBatchChecks` hand-back, OK report, first bracket, removed target: sets `LaterTargetsReturned`, returns `*LaterTargetRemovedError` before `BindHandles` and drift repair. Correct.
- `postBatchChecks` hand-back, second call: flag set, skipped, records as before. Correct.
- `postBatchChecks` hand-back, FAILED report: flag false, no refusal. Correct.
- `postBatchChecks` hand-back, `deltaErr` non-nil: returns earlier, so the extra `deltaErr == nil` is redundant but harmless.
- `RecordBatch` error path: the error is not `ErrCardNotDone`, so it returns `nil, err`. The batch stays non-terminal and the flag is set on `bs`. Correct.
- Once per bracket: `begin-batch` creates a fresh `BatchState` (`beginbatch.go:645`), so the flag resets on a re-drive. Correct.
- CLI branch: saves state, releases the lock, syncs fabric, then emits `later_target_removed` and the batch name. A save failure and a sync failure both surface in the message. Correct.
- `recover-batch`: it builds its own `BatchState` and never sets `HandBackRemovedTargets`, so it never refuses. Correct. I saw no `RecoverBatch` call-site change in the diff.

**Findings**
1. MEDIUM, `design`: `rename-leaves-test` misses tests in an external test package.
   - Location: `internal/planparser/validate.go`, in `checkRenameLeavesTest`, the `test.Unit != old.Unit` comparison.
   - Summary: under Go, an external test package has a distinct pseudo-unit (`doc.go:14-20`, `targetdirs.go:29`, `vocabname.go:57-62`). `golang-testing` makes `package foo_test` the default, so `unit#TestX` there never equals the renamed symbol's unit and the check stays silent. The incident class (tests beside the renamed symbol) is only caught for in-package tests. The fix is to compare through `goPackageDir`/`RefScope`, as the sibling checks do.
   - Evidence: the unit comparison in the `validate.go` diff, and the pseudo-unit mapping in `targetdirs.go:29,167`. I did not read `ParseRef` itself, so confidence is moderate.

2. LOW, `design`: `RemovedTargets` indexes pair `New` sides and Create targets through `targetCards`.
   - Location: `internal/planglyph/resolve.go:43-46`, used by `RemovedTargets`.
   - Summary: a later card whose Rename pair `New` side or Create target equals a symbol the batch deleted would draw the "removed a name the card targets" refusal, although the card creates that name. This is unlikely, but the refusal text ("keep the name") would mislead.
   - Evidence: `resolve.go:36-47`.

**Tool calls**
1. `grep BatchState{` in `internal/websterengine`, `grep Unit` in `internal/planparser/ref*.go`, and `targetCards` in `planglyph`: confirmed the flag resets on re-begin, and read the `targetCards` index. The `ref*.go` glob matched nothing, so that part returned no result.
2. `grep _test` in `internal/planparser/glyph*.go` and `ref*.go`, plus a search for `func ParseRef`: located it in `glyphref.go`. The earlier glob matched nothing.
3. `grep external|pseudo` in `internal/planparser`: confirmed the external-test pseudo-unit exists. I did not read `ParseRef` itself.
