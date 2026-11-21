```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's code changes for correctness (fix-code-correctness lens). Findings:

1. **MEDIUM** — `newFindings` in `internal/burlerengine/roundcheck.go` (F13 fix): the refusal for a second `origin: webster` finding cannot be satisfied by its own way forward.
   - The counter increments for every new BLOCKING webster finding under a halt reason, including one on a changed file that would otherwise pass.
   - The way forward says "with %q located at the changed line that causes it, or without it". A finding that keeps `origin: webster` is refused again, whatever its location.
   - The wording should tell the reviewer to drop `origin: webster` from the second finding, or to remove it.
   - The review asked to admit one exempt finding outside the changed files, not to cap all webster-origin findings. The new test row only covers the case where both findings sit outside the changed files.

2. **LOW** — the F11 coverage gap is only partly closed.
   - The new tests mutate-check `postBatchInputs` in `recordbatch.go`, which `recover-batch` shares, and `weaverengine/record.go:170` via `DroppedCreateTargetSkipsItsDoneCheck`.
   - The third site the review named, `weaverengine/resume.go:448`, has no test with a decision record that drops a Create. Deleting its `DecisionRecordPath` field keeps every test green.

3. **LOW** — F16 diverges from the review's `fix` key, so the fixer-report should mark it `fixed-differently`.
   - `validate.go` reads the plan segment's `run_subdir` through `loomrecipe.BouncerRunSubdir(loomshed.NamePlanBouncer)`. The review asked for the Plan-Burler-Review row's `run_subdir`.
   - This holds only while the Bouncer and review rows share a value, which the recipe comment states.
   - If the Bouncer row is missing, `found` is false and the round checks are silently skipped.

I traced these and found no defect:
- **F1:** `withoutDroppedCreates` filters group and card declarations, and the caller's cards stay untouched. `bindFakeIndex` fails the new handle test when BindHandles gets the unfiltered cards.
- **F2:** the wrapped read error is not `ErrCardNotDone`. Both webster verbs and the weaver return it as the fix's `way forward:` message. The test's directory-as-record input reaches that wrap. The `DroppedCreates` heading-parse error is unreachable through `PostBatchChecks`, so only the read error is tested.
- **F3:** `DroppableCreate` is set only for `create-not-done`. `createTargetWayForward` has a single caller, `failCardNotDone`, which `record-batch` and `recover-batch` share. Both new way-forward rows fail without the change.
- **F4:** the early return is gone, so carried amendments render marked, with the paragraph. The new row fails on the old code.
- **F5:** `containsToken` terminates and handles boundaries, and the plan-directory and `.md` checks work. Both new rows would have been admitted before.
- **F8, F12, F14, F15:** nothing to run. The F12 and F14 test edits are text-only assertions or markers, and F15 is help text.
- **F9:** the spawned-reviewer, probed-and-resumed-under-a-differing-fan and no-directory cases of `burlerreview_halt_test.go`, plus the recipe row test, cover the surviving mutants at `burlerreview.go:360`, `379` and `383`.

No tool calls beyond my context.
