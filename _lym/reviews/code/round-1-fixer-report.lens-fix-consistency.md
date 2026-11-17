```yaml
lens: fix-consistency
outcome: reported
```

**Task:** check the fix's change set for places where a changed statement and its twin elsewhere now disagree.

No twin was missed. One sentence the fix left behind now reads wrong:

- **LOW** — `internal/testcover/mutantverdict.go:63`, the `JudgeMutants` doc line "It is not judged either when its blocks hold no mutant, or it killed none and no run of it was unjudged."
  - After F1, a candidate whose only unjudged own run is of a mutant a covering test killed has an empty `Killed` and an empty `Unjudged`. It gets `ReasonKillsNone`, even though a run of it was unjudged.
  - The new lines 64-65 say that mutant is left out of all three lists, which is right. But line 63 still implies any unjudged run keeps the candidate judged, so the two lines disagree.
  - Evidence: the switch at lines 107-108 (`len(Killed)==0 && len(Unjudged)==0` gives `ReasonKillsNone`) and the F1 branch at lines 88-91, which skips the append when `coveringRunIn(kills)`.
  - A fix would reword line 63 to "…or it killed none and no answer of it rests on an unjudged run."

I checked the other places that state the F1, F2 or F3 behaviour and they agree with the new behaviour:
- the `MutantVerdict.Unjudged` field doc
- the `runMutants` doc
- the `gatecli/doc.go` lines at 1196 and 1222
- `testcover/doc.go:54-56`
- the `gateslot/doc.go` and `template.yaml` wording
- the `Long` help of `lym gate redundant`
- the integration-test run-count comment and constants

A repo-wide grep for "counts as a kill" and "own run" found no stale statement left outside the edited files.

Tool calls: one `grep` over the Go, markdown and yaml files for "own run", "unjudged" and "as a kill" outside tests. My context did not show whether any file I had not read repeats those statements.
