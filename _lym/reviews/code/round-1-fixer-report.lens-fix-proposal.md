```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the fixer's carried-out proposals (F1–F5) for a concrete case where each is wrong.

**Findings**

1. **MEDIUM** — F2, `internal/websterengine/suspect_test.go:191` with `export_test.go:97`.
   - Summary: the proposal's premise is false, so the way-forward wiring is still untested.
   - Evidence: the review says `suspect_test.go` asserts the reason text covering the way forward. The only caller of `checkRecoveredSuspects` is `TestCheckRecoveredSuspects_EmptyStartHoldsNothing`. It asserts `len(reasons) == 1` and `strings.Contains(reasons[0], "at tracked.txt")`. It never checks for "recover-batch 01" or for the `planPathClause` text.
   - Effect: replacing `planPathClause(...)` in `recoveredSuspectReasons` (`runplan.go:163`) with `""` or a wrong verb would still pass every test.
   - What the change did: it moved the argument into a production function and aliased the test seam to it, so the wiring is now shared. The wiring is still not asserted.
   - Whether the reason carries the way forward at all depends on `runplan.CheckRecoveredSuspects` (`suspect.go:332`). That function is not in my context, so I did not check it. If the reason for a copy at a tracked path includes the clause, a `strings.Contains(reasons[0], "lym webster recover-batch 01")` assertion in that existing test would close the gap.

**Proposals that held up**

- F1: the closure returns the same `"the plan changed since state.json was created; "` text as `runlevel.go:575`.
  - The runplan wrap still carries both fingerprints, so webster's refusal reads as it did before.
  - `batchCardEditError` is unaffected.
  - No file keeps the old "run state was created" phrase.
  - A shared caller gets no "the plan changed" clause unless it passes its own. The updated doc comment states this.
- F3: the new doc text matches the two pinned sites the review cited.
- F4: no file references the old heading "No shared substrate".
  - The retitled section still says webster imports no other loop's parser, report schema or digest contract. That sentence conflicts mildly with the section's own claim that webster shares `runplan`'s `Report` and `Digest`, but the fix did not introduce it.
- F5: the comment-only renames are correct. `runplan.ChangedPlanFiles` and `runplan.BatchIdentity` both exist.

**Tool calls**
1. Read `suspect_test.go` 170–220: my context lacked the test's assertions, which F2's premise depends on.
2. grep for the old heading and the F2 symbols across `.go` and `.md` files: my context lacked the other references to the retitled heading and the renamed seam.
